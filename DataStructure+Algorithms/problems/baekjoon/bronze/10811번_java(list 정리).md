https://www.acmicpc.net/problem/10811

## 문제 
- 바구니를 N개 가지고 있음 M번째 바구니의 순서를 역순으로 만들어야 함.역순으로 만들 범위를 정하고 그 범위에 있는 바구니를 역순으로 만듬. 역순으로 만들고 값 출력해라.
## 처음 든 생각
리스트의 범위를 정하는 기능을 사용해 역순으로 만들어 출력하면 된다.

1. 리스트를 만들고 범위를 정한다.
2. 리스트에서 범위의 값만을 뽑아내 역순으로 만들고 다시 제자리에 넣는다.


    List<Integer> bucket = new ArrayList<>();
    List<Integer> temp;
    ....
    Collections.reverse(temp);
    for(int k = 0; k< temp.size(); k++){
        bucket.add(<인덱스 값>,temp.get(k))
    }

## [문제 발생] 

ConcurrentModificationException이 발생함.
이유는 ArrayList의 서브리스트를 수정하는 동안 원래 리스트에 변경을 가했기 때문.
Collections.reverse(temp);로 서브리스트를 역순으로 만들고, 그 후에 bucket에 다시 추가하려고 할 때 원래 리스트가 변경되면 이 예외가 발생합니다.

## [해결]

수정할 인덱스에 직접 접근하기하면 된다.서브리스트를 역순으로 만든 후, 원래 리스트의 해당 범위에 직접 값을 넣도록 수정했다.

    public class Main {
        public static void main(String[] args) throws IOException {

            BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
            StringTokenizer st = new StringTokenizer(bf.readLine());

            int n = Integer.parseInt(st.nextToken());
            int m = Integer.parseInt(st.nextToken());


            List<Integer> bucket = new ArrayList<>();
            List<Integer> temp;// bucket의 reverse를 잠시 담아주는 값

            for (int i =1; i<n+1; i++) {
                bucket.add(i); //리스트를 만든다.
            }


            for (int g=0; g<m; g++){
                st = new StringTokenizer(bf.readLine());
                int i = Integer.parseInt(st.nextToken());
                int j= Integer.parseInt(st.nextToken());
                temp=bucket.subList(i-1,j);
                Collections.reverse(temp);
                for (int k = 0; k < temp.size(); k++) {
                    bucket.set(i - 1+k, temp.get(k));
                }
            }
            for(int i: bucket){
                System.out.print(i+" ");
            }

        }
    }


## [배운점]

아마 더 간단한 방법으로도 풀 수 있었을 것이다. 하지만 좀 복잡하더라도 리스트 사용에 익숙해지고 싶었다.
리스트의 사용법 약간 숙지
subList/Collections.reverse(list값)/list명.set()/.get()

bucket은 java.util.ArrayList instance을 저장하고 있는 힙 객체 주소를 참조한다. 
두번째 for문에 들어갔을때 temp도 참조변수로 스택에 쌓였고 반복문이 마치자 stack에서 사라진다. 
이후 마지막 반복문에서 i가 bucket을 돌며 값을 출력한다. 다시 말하지만 bucket내부에는 주소뿐이다. 주소를 타고 힙으로 들어가 저장된 값을 가져오는 것이다.

--- 

[subList참고](https://adjh54.tistory.com/190)

[Collections.reverse참고](https://hianna.tistory.com/570)

[list기본명령어,add,get,set..](https://hianna.tistory.com/568)