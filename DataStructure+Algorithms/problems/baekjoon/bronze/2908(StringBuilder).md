
## 문제 정의

2개의 상수 입력값이 있다. (길이값이 동일)
단, 이 입력값을 거꾸로 읽는다. 그 중에 큰 값을 출력한다.

## 처음 든 생각

숫자값으로는 reverse하기 힘들다. 

String으로 입력 받고 reverse한 값을 다시 Ineger로 변환해 대소 비교한다. 

## 문제 발생

String을 Integer형식으로 바꾸는 방법

## 해결

    public class Main {
        public static void main(String[] args) throws IOException {

            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

            StringTokenizer st = new StringTokenizer(br.readLine());

            String a= st.nextToken();
            String b = st.nextToken();


            String temp = "";
            String temp2 = "";
            for (int i = a.length()-1; i >= 0; i--) {
                temp += String.valueOf(a.charAt(i));
                temp2 += String.valueOf(b.charAt(i));

            }
            int n1 = Integer.parseInt(temp);
            int n2 = Integer.parseInt(temp2);

            if(n1 > n2){
                System.out.println(n1);
            }else{
                System.out.println(n2);
            }
        }
    }

## 개선점

문자열을 거꾸로 순환하는 것도 좋지만 더 간단한 방법이 있을 것.
StringBuilder에 reverse라는 기능이 있다. 

    int revA = Integer.parseInt(new StringBuilder(a).reverse().toString());

이런 형태로 사용 가능함.

## 배운점

이렇게 문자열이 자주 변해야 하는 곳에서 String을 사용하기 보다 StringBuilder를 사용하면 훨씬 효과적이다.

reverse,append,replace,insert가 가능함.

https://hianna.tistory.com/905
참고 내용