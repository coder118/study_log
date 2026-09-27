# 문제 정의

### 제약 조건 분석

* 게임 참가 인원 m은 2 이상 100 이하의 정수
* 튜브의 순서 p는 1 이상 m 이하의 정수
* 10부터 15까지의 숫자는 대문자 A부터 F로 표기

---

### 패턴 단순화 및 추상화

* 0부터 차례대로 숫자를 n진법 문자열로 변환하여 하나의 긴 전체 문자열로 연결
* 전체 문자열의 길이가 최소 t 곱하기 m 이상이 될 때까지 숫자 변환 계속 진행
* 연결된 전체 문자열에서 튜브의 순서인 p번째 인덱스부터 시작하여 m의 간격으로 문자를 추출하여 결과 구성

---

### 경계 조건

* 튜브의 순서 p가 마지막 순서인 m과 같은 경우 인덱스 연산 시 나머지 값이 0이 되는 처리 확인
* 필요한 전체 문자열 길이가 채워지면 즉시 숫자 변환 및 문자열 추가 중단 진행
* 10 이상의 숫자에 대해 대문자 알파벳으로 변환되는 로직 확인

---

### 최적의 알고리즘

1. 0부터 시작하여 숫자를 n진수로 변환하는 반복문 실행 진행
2. 변환된 n진수 문자열을 하나의 누적 문자열에 지속적으로 추가 진행
3. 누적 문자열의 길이가 t 곱하기 m 이상에 도달하면 변환 반복 중단 진행
4. 누적 문자열에서 인덱스 p 마이너스 1부터 시작하여 m 간격으로 문자를 선택하여 정답 문자열 생성 진행
5. 정답 문자열의 길이가 t에 도달하면 최종 결과 반환 진행

# 코드

    class Solution {
        public String solution(int n, int t, int m, int p) {
            StringBuilder fullString = new StringBuilder();
            StringBuilder answer = new StringBuilder();
            
            int number = 0;
            while (fullString.length() < t * m) {
                fullString.append(Integer.toString(number, n).toUpperCase());
                number++;
            }
            
            for (int i = 0; i < t; i++) {
                answer.append(fullString.charAt(i * m + (p - 1)));
            }
            
            return answer.toString();
        }
    }
