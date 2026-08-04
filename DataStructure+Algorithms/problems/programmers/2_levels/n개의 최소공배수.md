# 문제 정의


두 수 a와 b의 최소공배수는 a와 b의 곱을 a와 b의 최대공약수를 나눈 것과 같다는 성질을 가지고 있다

따라서 어떤 두 수의 최대공약수를 구한다면 최소공배수를 쉽게 구할 수 있는데 최대공약수를 구하기 위해 유클리드 호제법을 이용


# 코드

    class Solution {
        public int solution(int[] arr) {
            int answer = 0;

            if(arr.length == 1) return arr[0]; //원소가 한 개인 경우 바로 출력

            int g = gcd(arr[0], arr[1]); //처음 두 원소의 최대공약수
            answer = (arr[0] * arr[1]) / g; //처음 두 원소의 최소공배수

            /*  
            원소의 개수가 3개 이상인 경우
            앞의 두 원소의 최소 공배수와 다음 원소의 최소 공배수를 구하며
            배열의 끝까지 반복
            */ 
            if(arr.length > 2) { 
                for(int i = 2; i < arr.length; i++) {
                    g = gcd(answer, arr[i]);
                    answer = (answer * arr[i]) / g;
                }
            }

            return answer;
        }

        //최대 공약수
        private static int gcd(int a, int b) {
            int r = a % b;
            if(r == 0) return b;
            else return gcd(b, r);
        }
    }

1. 원소가 한 개인 경우에는 해당 원소를 바로 출력한다
2. 만약 원소가 3개 이상이라면 앞서 구한 최소 공배수와 다음 원소의 최소 공배수를 구하며 이를 배열의 마지막까지 반복

