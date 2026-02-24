
# 목차

1. [entity](#entity)
2. [dto](#dtodata-transfer-object)
3. [왜 분리해서 사용하는가?](#분리하는-이유)
------


![dto와 entity의 사용 전체 과정](/Spring/Spring_images/image2.png)



# entity

@Entity, @Column, @Id 등...

entity 클래스는 실제 DB테이블과 매핑되는 핵심 클래스이다. entity는 db의 **영속성의 목적으로 사용되는 객체**이기 때문에, request나 response값을 전달하는 클래스로 사용하는 것은 좋지 않다.

또한, 서비스 클래스와 비지니스 로직들이 entity클래스를 기준으로 동작하기 때문에 entity클래스가 변경되면 여러 클래스에 영향을 줄 수 있다.

> 중요한 점

entity에서 setter 메서드의 사용을 최대한 지양해야 한다.
이유는 위에서도 설명했듯이 이 값이 변경되면 영향을 미칠 클래스와 로직이 많기 때문에 최대한 변경을 최소화하여 객체의 일관성과 안정성을 보장하는 것이다.

> 그럼 어떤 것을 주로 사용할까?

setter대신 constructor(생성자) 또는 builder를 사용한다.

생성자로 초기화 하는 경우 불변 객체로 활용할 수 있고 불변 객체로 만들면 데이터를 전달하는 과정에서 데이터가 변조되지 않음을 보장할 수 있다.

> 코드 예시

    @Builder
    @Getter
    @Entity
    @NoArgsConstructor
    public class Membmer {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long idx;
        private String name;
        private String email;
    
        public Member(Long idx, String name, String email) {
            this.idx = idx;
            this.name = name;
            this.email = email;
        }
    }

    // 사용 방법
    Member member = Member.builder()
            .name("Jan")
            .email("Jan@Jan.com")
            .build();



[목차로](#목차)

# DTO(Data Transfer Object)


**계층 간의 데이터 교환을 위한 객체(java beans)**이며 JSON serialization과 같은 직렬화에도 사용되는 객체이다.

Controller 같은 클라이언트 단과 직접 마주하는 계층에서는 Entity 대신 DTO를 사용해서 데이터를 교환하며, 컨트롤러 외에 여러 레이어 사이에서 dto를 사용할 수 있지만 view와 controller 사이에서 데이터를 주고받을때 활용이 자주된다.

**순수한 데이터 객체를 가지고 있으며 getter,setter외에 비즈니스 로직은 포함되지 않는다.**

만약, dto의 값이 entity에서 필요할 경우, toEntity() 메서드를 통해서 DTO에서 필요한 부분을 이용하여 Entity로 만든다.

DTO는 원래 DAO(Data Access Object) 패턴에서 유래된 단어로 DAO에서 DB 처리 로직을 숨기고 DTO라는 결괏값을 내보내는 용도로 활용함.

> 코드 예시

    public class MemberDTO {

        pulic MemberDTO() { }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }


[목차로](#목차)

# 분리하는 이유

dto는 순수하게 데이터를 담고 있다는 점이 entity와 유사하지만, 목적 자체가 전달이고 읽고, 쓰는 것이 가능하고, 일회서으로 사용되는 성격이 강하다.

예> entity에서 getter해와서 원하는 정보를 표시할 수 있다. 하지만 복잡하게 엮인 데이터를 가져올때 어려운 경우가 종종있다. 그러다보면 필드나 로직을 추가해야 하는데 이렇게 되면 entity객체가 망가뜨려지게 된다.
하지만 dto는 entity의 값을 그대로 가지고 있는 일회성으로 필요에 맞게 수정을 할수 있다.



[목차로](#목차)

------------------
[참고 자료 1](https://velog.io/@pjy05200/DTO%EC%99%80-Entity-Class%EC%9D%98-%EC%B0%A8%EC%9D%B4)

[참고자료2](
https://wildeveloperetrain.tistory.com/101#google_vignette
)

[목차로](#목차)