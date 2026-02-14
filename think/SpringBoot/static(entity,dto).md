

### 26.2.14

# entity와 dto의 변환 과정

entity가 db와 깊게 연결되어 있어서 직접 값을 다루는 것에는 리스크가 있다. 그래서 dto라는 클래스를 만들어서 데이터를 다룬다.



dto를 entity로

List<dto>를 List<entity> 형태로 바꿔야 했다. 
그래서 dto 내에 entity로 변경하는 메서드를 만들었다.

    public  QuestionDTO fromEntity(Question question){
        QuestionDTO dto = new QuestionDTO(question);
//        dto.setId(question.getId());
//        dto.setSubject(question.getSubject());
//        dto.setContent(question.getContent());
//        dto.setCerateDate(question.getCreateDate());
        return dto;


위 메서드에 static의 필요할까?에 대한 생각이 들었다. 
jvm이 클래스를 로딩할때 해당 메서드를 메서드 영역에 미리 올려두게 되면 쓸데없이 메모리가 낭비되는게 아닐까 생각했었다. 

찾아본 결과 무분별한 사용일때 낭비가 되는 것이지 몇개로 영향을 미치지 못한다는 것과 코드의 가독성 측면에서도 쓰는게 좋다는 것을 알게 되었다.

    //static 없을때 
    QuestionDTO dto = new QuestionDTO(); 
    dto.fromEntity(question);

    //static 붙일때
    QuestionDTO dto = QuestionDTO.fromEntity(question); 

의미론적 설계라고도 한다. 위 코드는 외부의 상태를 가지고 참조해서 계산하는 것이 아니다. 그냥 단지 question을 dto라는 새로운 가방을 만드는 도구일뿐이다. 
이런 순수 함수 같은 것은 static을 써주는 것이 편하다.


