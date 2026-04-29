### 문제 발생 시점

어떤 부분에서 잘 못되었는지 그림으로 간단히 정리

![기존 엔티티 구조](https://velog.velcdn.com/images/ccoode/post/6dabba78-270c-4a0c-a7e6-6329aad5c360/image.png)

보면 알다시피 payment와 멱등성 테이블을 따로 분리를 해두었다. 해당 테이블을 만들때 생각했던 것은 결제가 되는 것과 멱등성을 관리하는 것이 같이 붙어있게 된다면 복잡성이 너무 증가할 것 같다라는 이유였다. 

하지만, 멱등성 테이블을 만들다 보니까 이것이 결국 결제 테이블의 내용이라는 것을 깨닫게 되었다. 

문제는 한 가지 더 있었다.
토스 페이먼츠를 사용할때 난 항상 같은 orderId를 사용하면 되는줄 알았다. 무슨 말이냐면, 한 주문에 해당하는 유일한 값인 orderId가 존재하고 그 값을 토스에 넣고 header에 멱등키만 바꿔서 넣어주면 되는 줄 알았다.
~~(핑계라면, 너무 시간이 촉박했다는 것.. )~~

여튼, 수정할 내용은 아래와 같다.

1. 멱등성 테이블을 결제 테이블로 통합. order와 payment의 관계를 1:N으로 수정.
2. 결제 승인을 해주는 기본 비즈니스 로직이 항상 같은 orderId를 사용하게 만들어두었다.
원래, **FAILED 상태가 되었을때 동일한 orderId를 사용하고 멱등키만 바꿔주는 로직이었다.** 해당 로직을, _**FAILED된 상태에서 사용자가 재시도 할 경우 새로운 orderId와 멱등키를 발급한다.**_
(참고로, 네트워크,서버에러의 경우 동일한 orderId와 멱등키를 사용하는 로직은 구현이 되어있다.)

# refactoring

## 1번 

#### 수정한 테이블 형태
![수정된 테이블 형태](https://velog.velcdn.com/images/ccoode/post/31d8d5b3-6732-4b14-af31-55416cc4c384/image.png)


#### 수정한 결제 상태 
```
public enum PaymentStatus2 {
    READY,
    PENDING,
    PAID,
    FAILED,
    UNCERTAIN
}
```

결제 상태 부분은 원래 결제 테이블의 상태와 멱등성의 상태를 따로 구분을 해서 구현을 해두었다. 그러다보니, 복잡도가 너무 증가하는 바람에 위와 같이 하나로 통합을 진행했다.


## 2번

### 기존 코드

```
  @Transactional(propagation = Propagation.REQUIRES_NEW)
    public IdempotencyRecord getOrCreateRecord3(String orderId, RequestType type, String originalOrderId) {
        // 1. 먼저 있는지 확인 (락 없이 가볍게)
        Optional<IdempotencyRecord> existingOpt = idempotencyRepository.findByOrderIdAndType(orderId, type);

        if (existingOpt.isPresent()) {
            // 2. 있다면 락을 걸고 다시 조회하여 상태 체크
            IdempotencyRecord existing = idempotencyRepository
                    .findByOrderIdAndTypeForUpdate(orderId, type)
                    .orElseThrow(); // 위에서 확인했으므로 무조건 있음

            return handleExistingRecord(existing, orderId);
        }

        try {
            // 3. 없다면 새로 생성
            String initialKey = generateTossKey(orderId);
            IdempotencyRecord newRecord = (type == RequestType.PAYMENT)
                    ? IdempotencyRecord.createPayment(orderId, initialKey)
                    : IdempotencyRecord.createCancel(orderId, initialKey, originalOrderId);

            return idempotencyRepository.saveAndFlush(newRecord);
        } catch (DataIntegrityViolationException e) {
            // 그 사이 누군가 인서트했을 아주 희박한 확률을 위한 방어 코드
            return idempotencyRepository.findByOrderIdAndTypeForUpdate(orderId, type)
                    .map(record -> handleExistingRecord(record, orderId))
                    .orElseThrow(() -> e);
        }
    }
```

```
// 상태 분기 로직 공통화
    private IdempotencyRecord handleExistingRecord(IdempotencyRecord existing, String orderId) {
        switch (existing.getStatus()) {
            case SUCCESS: return existing;
            case UNCERTAIN:
                // 네트워크 에러 등으로 상태가 불분명했던 경우:
                // 토스 가이드에 따라 '동일한 Idempotency-Key'를 유지하며 재시도
                // (이미 성공했을 수도 있으므로 키를 바꾸면 중복 결제 위험이 있음)
                existing.markAsPending(); // 다시 PENDING으로 돌리고 진행
                return existing;
            case PENDING:
                throw new BusinessException(ALREADY_PROCESSED_PAYMENT);
            case FAILED:
                //todo 여기서 오류를 던져서 프론트에서 다시 결제하기 버튼을
                // order API와 연동해 새로운 orderId를 생성하는 방향으로 가야 한다.
                String newKey = generateTossKey(orderId);
                existing.retry(newKey);
                return existing;
            default: throw new BusinessException(ALREADY_PROCESSED_PAYMENT);
        }
    }
```

#### 1. 선조회 

먼저 DB에 해당 주문(orderId)과 작업 유형(type)으로 기록된 멱등성 레코드가 있는지 락(Lock) 없이 가볍게 조회한다.

#### 2. 상태 확정

조회된 레코드가 있다면, 즉시 SELECT ... FOR UPDATE를 사용하여 해당 로우에 락을 건다. 
비관적 락을 건 이유는 여러 요청이 동시에 들어왔을 경우 상태가 2번 업데이트되는 문제를 막고 추후에 있을 중복 결제 처리를 사전에 방지하기 위해서이다.

- SUCCESS: 이미 완료된 결제이므로 기존 결과를 즉시 반환.

- UNCERTAIN: 네트워크 에러 등으로 결과가 불분명한 상태. 토스 가이드에 따라 기존 키(Idempotency-Key)를 유지하며 재시도하여 중복 결제를 방지.

- PENDING: 현재 결제가 진행 중임을 의미. 만약 pending중에 동일한 키가 들어오면 예외처리.

- FAILED: 실패한 요청은 _**새로운 멱등키만 갱신해서**_ 다시 결제 기회를 부여.

#### 3. 동시성 예외 방어

조회된 기록이 없다면 새로운 IdempotencyRecord를 생성.

- 유니크 제약조건을 활용해서 동시에 INSERT를 시도할 경우, 한쪽은 유니크 제약 위반으로 에러(DataIntegrityViolationException)가 발생하게 만듬.

### 수정 코드

```
 @Transactional(propagation = Propagation.REQUIRES_NEW)
    public Payment2 getOrCreatePaymentAttempt(Order order, RequestType type, String originalOrderId) {

        try {
            // 1. 최신 레코드 조회 (여러개가 있을 수 있다.)
            Optional<Payment2> latestOpt = paymentRepository2.findFirstByOrderOrderByCreatedAtDesc(order);

            if (latestOpt.isPresent()) {
                Payment2 latestPayment = latestOpt.get();

                // 비관적 락으로 해당 행 점유
                paymentRepository2.findByIdForUpdate(latestPayment.getId());

                Payment2 result = handleExistingPayment(latestPayment);
                if (result != null) return result; //paid, pending, uncertain일때
            }

            // 2. 신규 생성 (FAILED 이후 혹은 최초 생성)
            return createNewPaymentAttempt(order,type);

        } catch (DataIntegrityViolationException e) {
            // 동시성 이슈: 거의 동시에 두 스레드가 신규 생성(v1 등)을 시도했을 경우
            log.warn("결제 레코드 생성 중 경합 발생 - 최신 데이터 재조회");
            Payment2 latest = paymentRepository2.findFirstByOrderOrderByCreatedAtDesc(order)
                    .orElseThrow(() -> e); // 여전히 없다면 원본 에러 던짐

            return handleExistingPayment(latest);
        }
    }
    private Payment2 createNewPaymentAttempt(Order order,RequestType type) {
        // v1, v2... 접미사 생성을 위한 count
        int attemptCount = paymentRepository2.countByOrder(order);

        String baseOrderNumber = order.getOrderNumber();
        String tossOrderNumber = (attemptCount == 0) ? baseOrderNumber : baseOrderNumber + "_v" + (attemptCount);
        String tossIdempotencyKey = generateTossKey(baseOrderNumber);

        Payment2 newPayment = Payment2.createPayment(
                order,
                tossOrderNumber,
                order.getTotalAmount(),
                tossIdempotencyKey,
                type //payment 승인, 혹은 cancel 환불
        );

        return paymentRepository2.saveAndFlush(newPayment);
    }
```
```
private Payment2 handleExistingPayment(Payment2 curPayment) {
        switch (curPayment.getStatus()) {
            case PAID: return curPayment;
            case UNCERTAIN:
                // 네트워크 에러 등으로 상태가 불분명했던 경우:
                // 토스 가이드에 따라 '동일한 Idempotency-Key'를 유지하며 재시도
                // (이미 성공했을 수도 있으므로 키를 바꾸면 중복 결제 위험이 있음)
                curPayment.markAsPending(); // 다시 PENDING으로 돌리고 진행
                return curPayment;
            case PENDING:
                throw new BusinessException(ALREADY_PROCESSED_PAYMENT);
            case FAILED:
                // null 반환 시 상위 메서드에서 createNewPaymentAttempt() 호출
                return null;
            default: throw new BusinessException(ALREADY_PROCESSED_PAYMENT);
        }
    }
```

#### 1. 주문별 최신 결제 시도 조회

먼저 해당 주문(Order)에 대해 기존에 시도했던 결제 기록이 있는지 확인한다.

만약, 존재한다면 해당 값에 락을 걸어서 동시에 같은 주문이 처리되지 않게 막는다.

#### 2. 상태 확정(-> 상태 기반 의사 결정)

조회된 최신 결제 건의 상태에 따라 '기존 건을 유지할지' 아니면 '새로 만들지' 결정한다.

- PAID (성공): 이미 결제가 완료된 건. 중복 승인을 막기 위해 기존 데이터를 즉시 반환하고 종료.

- UNCERTAIN (불확실): 토스 API 호출 중 타임아웃 등으로 결과를 모르는 상태. 토스 가이드에 따라 동일한 Idempotency-Key와 orderNumber를 유지하며 재시도해야 합니다. (키를 바꾸면 중복 결제 위험이 있기 때문입니다.)

- PENDING (진행 중): 누군가 이미 결제를 진행 중. 중복 요청 방지를 위해 예외처라.

- FAILED (실패): 결제가 실패. 이 경우 해당 시도는 끝난 것이므로, 신규 생성으로 넘어가기 위해 null을 반환합니다.

#### 3. 실패/최초 시도 시 신규 버전 생성

결제가 처음이거나, 이전 시도가 실패(FAILED)한 경우 완전히 새로운 Payment2 객체를 생성한다.

_**사실상 해당 부분이 잘 못되어서 전체를 리팩토링하고 있는 것인데, 왜 이렇게 바꾸게 되었냐면, 
토스 API는 동일한 orderId로 재시도할 경우 이전 성공/실패 여부에 따라 요청을 거부할 수 있기 떄문이다.**_

- **해결책 **: countByOrder를 통해 몇 번째 시도인지 확인한 후, ORD-123_v1, ORD-123_v2와 같이 주문번호 뒤에 버전(v)을 붙여 새로운 시도를 기록. 이때 멱등성 키(Idempotency-Key)도 새롭게 발급한다.

#### 4. 동시성 예외 방어

기존 코드와 동일한 내용. 

---
위에서 1차 수정을 마치고 이제 FAILED된 요청이 다시 시도할때 토스 API에 버저닝된 값이 들어가게 만들어줘야 한다.

```
        Payment2 currentPayment = paymentStatusService.getOrCreatePaymentAttempt(order, RequestType.PAYMENT,null);

        // 2. 이미 성공한 요청이면 저장된 응답 반환
        if (currentPayment.getStatus() == PaymentStatus2.PAID) {
            return paymentStatusService.parseResponse(currentPayment.getResponseBody());
        }

        ConfirmRequest tossRequest = new ConfirmRequest(
                request.paymentKey(),
                currentPayment.getOrderNumber(), // 이 부분이 v1, v2 등으로 바뀜
                request.amount()
        );

        String tossIdempotencyKey = currentPayment.getIdempotencyKey();
        headers.set("Idempotency-Key", tossIdempotencyKey+ (testCode != null ? UUID.randomUUID() : ""));
        headers.set("Authorization", "Basic " + encodedKey);
        headers.setContentType(MediaType.APPLICATION_JSON);

        HttpEntity<ConfirmRequest> entity = new HttpEntity<>(tossRequest, headers);

```
위의 코드에서 tossRequest를 만들어주었다. 원래는 들어온 처음 들어온 confirmRequest를 사용해서 entity에 값을 넣었었다. (항상 같은 orderId를 사용)

하지만, 이제 FAILED될 경우를 생각해서 현재 결제 기록에서의 orderId(코드에서는 orderNumber)를 새롭게 넣어주는 것이다.

---
리팩토링을 하면서 구조 자체를 바꾸는 것이다 보니까 막상 손을 댈려고 하니까 막막했다. 그래서 단계적으로 접근하는 것이 중요한 것 같아서 크게 문제를 2가지로 나누고 기존 코드에서의 문제를 찾아내어 어떻게 수정을 할지 정리를 하면서 생각을 정리하니까 훨씬 수월하게 리팩토링을 마무리할 수 있었다.