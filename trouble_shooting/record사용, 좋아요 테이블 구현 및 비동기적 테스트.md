# 목차

1. [record 문제 해결](#record에서-생긴-문제-해결)
2. [좋아요 테이블 문제 해결](#좋아요-테이블-문제-해결)
3. [비동기적 실행 환경으로 테스트하기](#비동기적-테스트-코드)


# 1. Record 문제 설명

점프 투 스프링 부트라는 책을 공부하면서 생긴 일이다.
<br>
게시글을 수정할 떄 사용자가 수정하기 편하게 기존에 적혀있는 내용을 그대로 보여줘야 했다.
<br> 
책에서는 form 클래스를 만들어서 값을 받고 set으로 수정하는 방식을 진행했다. 객체가 생성된 후에도 언제 어디서든 값이 변할 수 있다. 규모가 커진 프로젝트에서는 "누가 여기서 값을 바꿨지?"라는 추적 비용이 발생할 수 있다. 
<br>

    @PreAuthorize("isAuthenticated()")
    @GetMapping("/modify/{id}")
    public String questionModify(QuestionForm questionForm, @PathVariable("id") Integer id, Principal principal) {
        Question question = this.questionService.getQuestion(id);
        if(!question.getAuthor().getUsername().equals(principal.getName())) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "수정권한이 없습니다.");
        }
        //set으로 폼 내부의 값을 바꿀 수 있다. 
        questionForm.setSubject(question.getSubject());
        questionForm.setContent(question.getContent());
        return "question_form";
    }

그래서 record를 사용해봐야겠다고 생각했다.

## record에서 생긴 문제 해결

record는 불변 객체라서 한 번 정해진 값을 set해서 바꿀 수 없다는 것이다. 
<br>
하지만 지금 나의 경우는 특정한 값을 form (=record)안에 넣어야 하는 상황이었고 그러기 위한 해결책은 set이 아니라 new로 만들어주는 방식이었다.
<br>

하지만,**new 로 바로 만들어주면 유지보수할때 문제가 생길 수 있다.** 그래서 new를 바로 컨트롤로에 연결하는 방식보다 외부에서 만든 new를 DI받는 방식으로 구현했다.


    public record QuestionForm(
            @NotEmpty(message="제목은 필수항목입니다.")
     ....
    ) {

            public static QuestionForm form(Question question) {
                    return new QuestionForm(question.getSubject(), question.getContent());
            }

    }

    //컨트롤러 코드
    QuestionForm questionForm = QuestionForm.form(q);
    model.addAttribute("questionForm", questionForm);//모델로 전송해준다.

이렇게 되면 설계 측면에서 의존성이 존재하지 않지만 실행시 런타임 의존관계가 형성되면서 다이나믹한 관계를 설정할 수 있게 된다. 

# 2. 좋아요 테이블 문제 설명

마찬가지로 점프 투 스프링 부트를 공부하면서 생긴 문제이다.

    @ManyToMany
    Set<SiteUser> voter;

책에서는 위와 같이 ManyToMany 방식을 사용했다. 하지만 이 방식은 누가 좋아요를 클릭했는지 알 수 없고 중간 테이블을 직접 관리할 수 없어서 확장이 불가능한 문제가 있다.

## 좋아요 테이블 문제 해결

새로운 Likes 테이블을 만들어 줬다. (**Like라는 클래스로 이름을 지으면 오류가 발생한다.** 가끔 나오는 실수라고 한다.)

    @Entity
    @Getter
    public class Likes {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private int id;

        @ManyToOne
        Question question;

        @ManyToOne
        SiteUser siteUser;

        public void QuestionLike(Question question, SiteUser user) {
            this.question = question;
            this.siteUser = user;
        }
    }

하나의 질문에 여러개의 좋아요가 만들어질 수 있고 한명의 유저가 여러개의 좋아요를 만들 수 있으므로 ManyToOne의 관계를 가진다. 

<br>

서비스 로직이 어떻게 짜여져야 하는지 고민했다. 사용자 한명은 하나의 질문에만 작성할 수 있다. 한마디로 **사용자 id와 질문 id가 합쳐진 Likes라는 객체는 유일한 값**이 된다. 그래서 like repository에 findBy로 질문과 유저를 검색하면 찾을 수 있는 함수를 구현했다.


    Optional<Likes> exsistinglike = this.likeableRepository.findByQuestionAndSiteUser(question, user);

여기서 존재한다는 것은 **DB 내부에 이미 있는 값**이라는 것이다. 그렇다면 이전에 눌러진 값이라는 말이 되므로 DB 내부에서 삭제해줘야 한다. **반대로**, 값이 존재하지 않는다면 Likes라는 객체 값을 새로 만들어서 DB에 저장해줘야 한다.

    if (exsistinglike.isPresent()) {
        
        this.likeableRepository.delete(exsistinglike.get());
    }else{
        Likes like = new Likes();
        like.QuestionLike(question,user);
        this.likeableRepository.save(like);
    }


-----

# 비동기적 테스트 코드

이렇게 문제를 해결해주다가 동시에 여러명의 사람이 동시에 누르게 되어도 이 코드가 잘 동작하는지 궁금해져서 간단한 테스트 코드를 작성해서 확인해보았다.

<details>

<summary>테스트 코드 확인</summary>
    
    
    @Test
    @DisplayName("50명이 동시에 좋아요를 누를 때 데이터 정합성 테스트")
    void threadTEst() throws InterruptedException {
        // 1. 준비 단계
        int threadCount = 50;
        // 50개의 스레드 풀 생성
        ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
        // 50개의 스레드가 모두 끝날 때까지 대기하기 위한 장치
        CountDownLatch latch = new CountDownLatch(threadCount);

        int len = this.questionService.getList().size() - 1; //가장 최신 글
        Question question = this.questionService.getQuestion(len);
        List<SiteUser> users = this.userRepository.findAll();

        // 2. 실행 단계: 50명이 비동기적으로 투표
        for (int i = 0; i < threadCount; i++) {
            final int userIdx = i;
            executorService.submit(() -> {
                try {
                    // 각 스레드에서 vote 실행
                    this.questionService.vote(question, users.get(userIdx));
                } catch (Exception e) {
                    System.err.println("에러 발생: " + e.getMessage());
                } finally {
                    // 실행이 끝나면 숫자를 하나 줄임
                    latch.countDown();
                }
            });
        }

        // 3. 모든 스레드가 끝날 때까지 대기
        latch.await();

        // 4. 검증 단계
        // 영속성 컨텍스트를 새로고침하거나 DB에서 직접 숫자를 세어와야 정확합니다.
        long finalCount = this.likeableRepository.countByQuestion(question);
        System.out.println(users.size());
        assertEquals(50,finalCount);

    }


</details>


문제없이 실행되었다. 원인을 찾아보니까 테스트 환경에서 사용되는 H2 데이터베이스가 락을 걸지 않아도 한 트랜잭션이 쓰기 시작하면 테이블 전체에 락을 건다고 한다. 
<br>
하지만 테스트환경이 아니라 mySQL을 사용하게 되면 위와 같이 동시에 실행되었을떄 문제가 될 가능성이 크다.
<br>

문제를 해결하기 위해선, 해당 로직이 실행될때 **비관적 락을 걸어주면 이 문제를 해결 할 수 있다**. 

    public interface LikeRepository extends JpaRepository<Likes,Integer> {

        @Lock(LockModeType.PESSIMISTIC_WRITE)
        Optional<Likes> findByQuestionAndSiteUser(Question question, SiteUser siteUser);
    ....

vote 내부에서 사용되는 Jpa에 비관적 락을 걸어줌으로써 여러 스레드가 동시에 들어와도 다른 스레드는 잠시 대기 상태에 있다가 한 스레드가 작업을 마치면 작업을 실행시키는 것이다.

