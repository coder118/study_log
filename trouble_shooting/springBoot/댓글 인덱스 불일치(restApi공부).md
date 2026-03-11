# 상황 설명


> 1번 파일

    @RequestMapping("/api/v2/posts/{postid}/comment")
    class <클래스 이름>
    ....
    ...

    @DeleteMapping("/{commentId}")
    @Transactional
    public RsData2<CommentDto> delete(@PathVariable("postid") int postid,
                          @PathVariable("commentId") int commentid) {

        Post post = postService.findById(postid).get();
        Comment comment = post.getComments().get(commentid);
        post.deleteComment(commentid);

        return new RsData2<>(
                "%d번 글의 %d번쨰 댓글이 삭제되었습니다.".formatted(postid, (commentid+1)),
                "204-1",
                new CommentDto(comment)
        );

    }
-------
> 2번 파일

    public void deleteComment(int id) {
        Comment comment = findCommentById(id).get();
        comments.remove(comment);
    }

    public Optional<Comment> findCommentById(int commentId) {


        return comments.stream()
                .filter(c -> c.getId() == commentId)
                .findFirst();
    }


포스트맨을 이용해 1번파일에 http://localhost:8080/api/v2/posts/2/comment/1을 delete 보내게 되면 Nosuchelement가 발생했고
<br>
 http://localhost:8080/api/v2/posts/1/comment/4를 실행시키면 outOfindex가 발생했다.

## 디버그

코드를 찬찬히 뜯어보니 실습 연습을 하면서 짜던 코드가 좀 꼬인것을 확인할 수 있었다.
<br>
지금 p1에는 3개의 댓글이 있었는데 각 댓글의 id는 1,2,3을 가졌다. 반면 댓글이 저장된 arraylist에는 인덱스가 0,1,2형태로 저장되었다. 
<br>
또한, p2에는 2개 댓글 id가 4,5로 존재했고 arrayList에 인덱스가 0,1로 존재했다.
그러다보니 값을 찾을 수 없고(=nosuch) 인덱스 오류(outOfindex)가 발생한 것이다.

## 해결책

id의 첫번째 값을 기준으로 잡은 후 남은 id에 첫번째 값을 빼주는 것이다.
<br>
- 1,2,3=> 1-1,2-1,3-1=>0,1,2라는 인덱스 형태가 나온다.
- 4,5=> 4-4,5-4 =>0,1이라는 인덱스 형태가 나온다.
---

    public Optional<Comment> getCommentByRelativeIndex(int commentId) {

        int firstCommentId = comments.get(0).getId();


        return comments.stream()
                .filter(c->{
                    int index = c.getId()-firstCommentId;
                    return index==commentId;
                })
                .findFirst();
    }

http://localhost:8080/api/v2/posts/2/comment/0 을 하면 p1의 첫번째 댓글이 삭제된다.
<br>
약간의 조건이 있다면 첫번째값을 1이 아니라 0으로 잡아야 한다는 것이다.
<Br>
만약, 프론트 협업시 편의를 위해서 1로 호출할 수 있게 하고 controller에서 -1처리를 해줘도 좋을 것 같다.



# 배운점
이 문제는 comment entity를 따로 만들어주지 않았을때 발생하는 예외적인 경우이다. 간단하게 실습하는 과정이라 이런 부분은 미쳐 생각하지 못했다. 다음엔 comment entity를 생성해서 댓글의 독립적인 id를 만들어주거나 댓글을 한 리스트에 저장할때는 인덱스와 댓글 id의 일치 문제가 있다는 것을 염두에 둬야 할 것 같다.