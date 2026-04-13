# 상황 설명

> ApiCommentController.class

public record commentReq(
            @Size(min=1,max=100,message = "aaaa")
            @NotEmpty
            String content
    ){}

    @PostMapping
    @Transactional
    public RsData2<CommentDto> create(@PathVariable("postid") int postid,
                                      @RequestBody @Valid commentReq commentReq) {
        
        Post post = postService.findById(postid).get();
        
       
        Comment comment =post.addComment(post,commentReq.content);
        
        
        // 이 부분이 문제가 발생한 지점이다. 
        return new RsData2<>(
                "%d번 글에 %d번째 댓글을 작성했습니다.".formatted(postid, comment.getId()),
                "200",
                new CommentDto(comment)
        );


-----
> Post.class


    @OneToMany(mappedBy = "post",
                cascade = {CascadeType.PERSIST, CascadeType.REMOVE},
                fetch = FetchType.LAZY,
                orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    public Comment addComment(String content) {
        Comment comment = new Comment(content, this);
        comments.add(comment);

        return comment;
    }

# 문제 정의

컨트롤러에서 comment.getId()와 comment를 출력하는 시점에서 nullPointerEx문제가 발생했다. <br>
처음에는 값이 안 넘어온것으로 생각했지만 값은 잘 넘어오고 있었다. 어떤 문제가 있는지 고민해본 결과

<br>

**트랜잭션 커밋이 끝나지 않고 호출을 해서 @EnableJpaAuditing이 동작하지 않으면서 BaseEntity(날짜, id값을 편하게 만들어주는 곳)의 값이 들어가지 않았다.** 라고 생각했다.

<br>
이때 생각했던 방법은 영속성 컨텍스트가 떠오르며, sql 지연쓰기에 값을 강제로 밀어넣어주면 DB로 insert되면서 실행되지 않을까 생각했다. 
<br>
아쉽게도, 문제가 해결되지 않았다.
<br>
<br>

> PostService.class

    public Post write(String title, String content) {
        Post post = new Post(title, content);
        return postRepository.save(post);
    }

post의 경우는 여기에서 잘 실행되고 있었고 차이가 안 느껴질때쯤. **save를 확인할 수 있었다.**

<br>
save가 호출되면 그때부터 JPA가 관리를 시작하면서 id,날짜와 같은 값이 채워져서 바로 사용할 수 있었던 것이다.
<br>

반면, comment에 실행되었던 코드는 단지 리스트에 값을 추가만 해준 경우인데 값이 채워질 수가 없었다. 하지만, flush를 실행하면 Post내부에 있는 CommentList에 값이 추가되는 거니까 될 것이라고 생각했지만, **JPA는** title과 content등이 바뀌지 않은채로 리스트 요소만 추가된 경우 값의 차이를 느끼지 못하고 **DB에 아무런 요청을 보내지 않을 수 있다.**

> comment의 id와 dto에 들어가는 값을 임시 제거했을때는 잘 실행되었다.

# 해결책

    //컨트롤러
    Comment comment =CommentService.writeComment(post,commentReq.content);

    //서비스
    public Comment writeComment(Post post, String content) {

        Comment comment = post.addComment(content);
        return commentRepository.save(comment);
    }

CommentRepositoy를 생성해서 save를 동작시켜서 nullPointerEx문제를 해결했다.


# 또 다른 해결책

위에서 flush를 사용했을때 post의 변경사항을 인지하지 못해서 DB로 insert작업이 일어나지 않는다고 했었다. 그때 내가 사용했던 코드의 형태는 아래와 같다.

    public void saveAndFlush(Post post) {
        postRepository.saveAndFlush(post);
    }

위 코드에서 post만 변경사항을 확인해서 DB로 보내는것이 불가능했던 것이다.

<br>

만약, **레포지토리 전체를 flush해준다면** DB로 insert를 보낼 것이고 그럼 자연스레 ID와 날짜 값이 생기면서 nullPointer문제가 해결된다.

    public void flush() {
        postRepository.flush();
    }


# 배운점

이 문제를 좀 더 깊게 파보기 위해서 save가 저장되는 시점에 디버깅을 통해 코드를 살펴보았다.
<br>
살펴보면서 흥미로웠던 점은 save라는 JPARepository에만 있는 함수가 HashSet으로 구성되어 내부에 존재하고 있다는 것이었다. 심지어 save라는 기능을 찾는 과정은 일일히 모든 JPARepository를 다 돌면서 찾아냈다.(RepositoryFragment.interface의 findMethods를 찾아보면 된다.)

<br>
이 외에도 EnableJPAEditng이 어떻게 어느 타이밍으로 들어가길래 flush로 DB로 넘어가지 않는지 궁금해서 찾아봤다.
<br>
QueryExecutorMethodInterceptor.class의 doInvoke라는 함수부터 시작된다. 이곳을 한번 돌고 나오면 id와 날짜 값이 채워져있었다.

<br>

> 동작과정 정리.. 

RepostitoryComposition.class의 invoke 함수가 오버로딩되며 동작된다. Invoke Method by resolving the fragment that implements a suitable method.
=>RepositoryMethodInvoker.class의 RepositoryFragmentMethodInvoker을 타고 들어간다.=>SimpleRepository 클래스에서 save라는 함수가 실행되고 Post를 entitymanger가 관리한다. => SessionImpl의 persist가 실행되면서 id와 날짜가 동작한다.

https://gemini.google.com/app/02ae547382f6f706?is_sa=1&is_sa=1&android-min-version=301356232&ios-min-version=322.0&campaign_id=bkws&utm_source=sem&utm_medium=paid-media&utm_campaign=bkws&pt=9008&mt=8&ct=p-growth-sem-bkws&gclsrc=aw.ds&gad_source=1&gad_campaignid=21109724830&gbraid=0AAAAApk5BhmqXtyng40vQcEC8SvVr1bL4&gclid=Cj0KCQjwgr_NBhDFARIsAHiUWr4XKetHxXGIOJ14vKkMbxM6HZX866K3FRSqn-2H3N2hFT4V1ph7JNUaAvLEEALw_wcB

[audit동작원리 참고](https://study0304.tistory.com/71)