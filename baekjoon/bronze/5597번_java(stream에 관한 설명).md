# 오늘 공부 내용
     public static void main(String[] args) throws IOException {

        BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));

        int[] arr = new int[30];

        for (int i = 0; i < 28; i++) {
            int num = Integer.parseInt(bf.readLine());
            arr[num-1]=1;
        }
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] != 1) {
                System.out.println(i+1);
            }
        }

    }
아주 간단한 내용의 코드이다. arr과 num-1이런 부분에 아직 익숙하지 않다. 
    public class Main {
        public static void main(String[] args) throws IOException {

            BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
            
            List<Integer> arr = new ArrayList<>();
            List<Integer> arr_all = new ArrayList<>();

            for (int i = 1; i < 31; i++) {
                arr_all.add(i);
            }

            for (int i = 0; i < 28; i++) {
                int num = Integer.parseInt(bf.readLine());
                arr.add(num);
            }
            List<Integer> newNonematchlist = arr_all.stream().filter(n -> arr.stream()
                    .noneMatch(Predicate.isEqual(n))).collect(Collectors.toList());

            for (int i : newNonematchlist) {
                System.out.println(i);
            }
            //https://haenny.tistory.com/389
        }
    }
위의 코드와 동일한 내용이다. 하지만 리스트를 사용했다. 두 리스트를 비교해서 겹치는 것을 제외하고 남은 값을 출력하게 구현했다.


### [문제]
백준 5597문제이다. 주어진 입력은 28자리로 정해져있고 중복되는 값이 없다. 출력은 총 30명 중에 입력되지 않은 2명의 번호를 구하는 것이다.
### [처음 생각]
총 2가지 생각이 났다.
1. 배열의 28가지 값을 넣고 if문으로 나올때 없는 값을 찾으면 되지 않을까? 하는 생각. 

2. 파이썬의 리스트가 생각나며 contain함수를 사용하여 배열 안에 값이 있는 지 없는지를 파악하면 될 것 이라고 생각했다.

### [문제 발생]
1. 28가지의 입력을 받는 것은 성공했지만 비교할 배열이 없다는 것을 알았다. 또한 30arr와 28arr의 비교를 어떻게 해야 할지 감이 오지 않았다. 

2. 자바 리스트에 익숙하지 않아서 어떻게 contain과 비슷한 기능을 낼 수 있을지 고민했다.  

### [해결]
1. 배열의 기본적인 특성을 잊었다. 배열에는 인덱스가 있고 처음 만들어질때 내부 값은 0으로 초기화 된다. 그럼 내가 입력하는 값을 인덱스 값으로 넣고 그 인덱스 값의 value값을 1로 넣어주면 마지막에 호출되지 않은 배열 value에는 0이 남아있을 것이다. 그리고 그 값을 호출하면 된다.

2. 사이트를 찾아가며 리스트의 문법을 찾았다. 30명의 리스트값과 28명의 리스트 각각을 비교하여 공통되지 않는 값만을 뽑아주는 기능이 있을 것이라 생각했다. 
stream.filter를 사용하여 문제를 해결했다. 
arr_all의 값이 stream으로 변환되어 arr과 비교한다. 조건은 n이 arr에 없는지 확인한다. 만약 없다면 반환되는 값을 collect로 잡아 새로운 리스트로 만들어낸다. 

### [배운 점]
Stream(),filter(),collect() 에 대해 알게 되었다.
데이터를 다룰 때 보통 for문과 Iterator문을 사용했다. 이런 방식은 너무 길고 알아보기 힘들고 재사용성도 떨어진다. 예를 들어 List를 정렬해야 할떄 Collections.sort()를 사용해야 하고, 배열을 정렬할때는 Arrays.sort()를 사용해야 한다. 이런 문제를 해결하기 위해 만든 것이 stream이다. 
stream은 데이터 소스를 추상화하고 데이터를 다루는데 자주 사용되는 메소드들을 정의해놓았다. 여기서 *추상화*했다는 것은 데이터 소스가 무엇이던 간 같은 방식으로 다룰 수있다는 것이다. 

예>

    String[] strArr ={"aaa","bbb"};
    List<String> strList = Arrays.asList(strArr);

위의 두 소스를 기반으로 하는 스트림은 아래와 같다.

    Stream<String> strStream1 = strList.stream();
    Stream<String> strStream2 = Arrays.stream(strArr);

원래 배열과 리스트를 출력하기 위해선 for문이나 Collections를 사용해야 했지만 stream은 그런 단점을 없앴다.

    strStream1.sorted().forEach(System.out::println);
    strStream2.sorted().forEach(System.out::println);