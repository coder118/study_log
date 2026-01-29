
## 문제
S = 알파벳 소문자로 이루어진 단어 
a~z가 나열되어있고 S에 주어진 인덱스의 위치를 a~z자리에 대신해라.
단, 없을 경우에는 -1을 나타내라

## 처음 든 생각

입력받은 문자열을 반복해서 돌린다. 
a~z 길이 만큼의 배열을 만든다. 단 -1로 다 초기화 되어있다. (파이썬의 딕셔너리 개념이 필요)

## 문제 발생

자바에서 딕셔너리의 개념을 모르겠다. 
중복체크하는 부분과 입력되는 인덱스 부분에서 문제가 생겼다. 


## 해결

처음 든 생각에 비해 계속해서 기능이 추가되니 코드가 복잡해진 것 같아서 다시 차근차근 정리했다. 

1. a가 -1에 매칭되고 b가 -1에 매칭되는 내용을 HashMap으로 구현했다. ["a":-1,"b":-1]이런 느낌.
2. check_list라는 리스트를 만들고 addUniquevalue라는 함수를 만듦. 해당 함수는 리스트 내부에 값이 없으면 값을 추가하고 있으면 false를 반환한다.
3.map.containsKey로 map에 입력받은 문자열이 있는지 확인하는 작업인데. 
=> (수정함) 지금 생각하니까 map에는 값이 항상 존재한다. 의미가 없는 문장이네. 그냥 중복 여부만 따지는 조건문으로 바꿈
4. 조건문 내부에서  map.put(check,i);를 이용해 check 문자열 자리에 i를 넣어줬는데, i로 넣어줘서는 안되고 alpabet의 인덱스 자리를 반환해줘야 한다. *즉 해시맵 키의 인덱스 자리를 반환해야 한다.* 
=>(수정) 아 아니다. 문제 정의를 잊고 있었다. S문자의 인덱스 자리를 해시맵에 반환하는건데 문제를 잊었다. 
해시맵에 집중할 것이 아니었다. 

    
    
        public class Main {
            public static void main(String[] args) throws IOException {
                BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));

                String S = bf.readLine();

                String[] alpabet = {"a","b","c","d","e","f","g","h",
                "i","j","k","l","m","n","o","p","q","r","s","t","u","v",
                "w","x","y","z"};

                HashMap<String, Integer> map = new HashMap<>();


                for (int i = 0; i < alpabet.length; i++) {
                    map.put(alpabet[i], -1);
                }


                for (int i = 0; i < S.length(); i++) {

                    String check = String.valueOf(S.charAt(i)); // char타입을 Stringg형태로 변환

                    if(map.get(check)==-1 ){ 
                        map.put(check,i);
                    } // -1일때만 입력되게 한다. 이럼 이전에 입력된 값이 있다면 -1이 아닐 것이기때문에 걸러진다.

                }
            
                for (int value : map.values()) {
                    System.out.print(value+" ");
                }

            }
        }

## 고민할 부분

개선점이 있다. String으로 영어 배열을 만들지 않고 해쉬맵을 사용하지 않고 고칠 수 있다. 


    for (char c = 'a'; c <= 'z'; c++) {
        int position = -1;
        for (int i = 0; i < input.length(); i++) {
            if (c == input.charAt(i)) {
                position = i;
                break;
            }
        }
        bw.write(position + " ");
    }

위와 같은 방법도 있다. 아스키 코드로 접근했다. a에서 z까지의 반복을 돈다. 단 a라는 아스키 코드가 input입력받은 문자열에 존재하면 조건문 실행.중복 부분도 a에서 z까지 한번만 반복해서 자연스레 해결.

## 배운점
새로운 것을 배울때 자꾸 새로운 것에 집착하게 된다. 집중해야 하는 것은 문제가 어떤 것인지 알아야 한다. 풀이가 복잡해질때 다시 문제 정의에 집중.

HashMap의 간단한 사용법
map은 키-값 쌍(key-value pair)으로 데이터를 저장하는 컬렉션이다. 
키는 고유해야 하며, 값은 중복을 허용한다.

HashMap,TreeMap,LinkedHashMap등이 대표적인 Map 구현체이다. 

    import java.util.HashMap;
    HashMap<String, Integer> map = new HashMap<>();

    //put 함수
    map.put("apple", 5);     // Key: "apple", Value: 5
    map.put("banana", 10);   // Key: "banana", Value: 10
    map.put("cherry", 7); 

    //get 함수
    System.out.println(map.get("apple"));  // 5
    System.out.println(map.get("pear"));   // null (존재하지 않는 키)

    //containKey함수
    if (map.containsKey("banana")) {
        System.out.println("Value: " + map.get("banana"));
    }

https://sxbxn.tistory.com/64

더 자세한 내용은 HashMap에 대한 따로 내용을 만들어야 겠다.
https://orange-makiyato.tistory.com/117
