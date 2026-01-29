
# 문제점   


    List<HashMap<String,String>> wise_List = new ArrayList<>();

    HashMap<String,String> wise_dic = new HashMap<>();

    wise_dic.put(wiseSaying,Author);
    wise_List.addFirst(wise_dic);

이런 식으로 리스트 안에 해쉬맵을 담고 싶었다. 하지만 원하던 그림이 아니었다. 
위의 코드를 실행했을때 파이썬의 이중 리스트를 생각했다. addFisrt를 사용하여 스택처럼 값이 쌓이게 구현할 수 있을 것이라 생각함.

    [[입력값2,입력값2],[입력값1,입력값1]] 


완전 다른 형태이다. 

일단 해쉬맵의 기본 형태를 보면 [[key,value],[key2,value2]]형태이다. 그런데 내가 위에 한 것은

    [[[key,value],[key2,value2]],[[key,value],[key2,value2]]...]

이런 식의 구조가 되는 것이다.

해쉬맵을 왜 리스트에 감싸려고 했느냐가 중요하다. 
해쉬맵에 들어간 값을 출력해보니 힙 형식으로 값이 저장되고 값을 스택처럼 출력하고 싶은데 불가능해 보였다. 그래서 리스트의 addFirst를 사용하면 괜찮지 않을까 싶었다.

----

# 해결책

위에서 말했듯 리스트 형태로 key와 value가 있는 값을 만들고 싶었다. 
클래스를 만들어서 해결했다.

    public class WiseSaying {

        int index =0;
        String wise_saying;
        String author;

        public WiseSaying(int i,String saying,String au) {
            this.index = i;
            this.wise_saying = saying;
            this.author = au;
        }

    }

    public static...main{
        ...
        ..
        List<WiseSaying> wise_List = new ArrayList<>();

        //새로운 객체를 만들어서 리스트에 저장한다. 스택 형식
        wise_list.addFirst(new WiseSaying(in,wiseSaying,Author));

        //리스트에 저장된 값 나열
        for (var inform : l) {
            System.out.println(inform.index+" / "+inform.wise_saying+" / "+inform.author);
        }
    }

HashMap도 결국 클래스이다. 그래서 직접 클래스를 만들어서 <>에 넣어줄 수도 있다. 