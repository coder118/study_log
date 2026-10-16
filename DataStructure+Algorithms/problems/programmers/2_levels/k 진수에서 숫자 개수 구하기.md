# 문제 정의

### 제약 조건 분석

* 입력값의 크기에 따른 진수 변환 결과의 길이를 파악하고 자료형 범위를 검토 완료.
* 문자열 분리 과정에서 발생할 수 있는 빈 값의 예외 상황을 분석.

### 패턴 단순화 및 추상화

* 진수 변환된 문자열을 영을 기준으로 분리하여 소수 후보를 추출하는 방식을 도출.
* 추출된 후보가 숫자인 경우에만 십진수 정수로 변환하여 소수 판정을 수행하는 구조 설정.

### 경계 조건

* 연속된 영으로 인해 빈 문자열이 생성되는 경우를 걸러내는 예외 처리를 설정.
* 일은 소수가 아니므로 판정 대상에서 제외하는 조건을 검증.
* 변환된 숫자가 클 수 있으므로 제곱근까지만 나누어 판정하는 최적화 조건 완성.

### 알고리즘 설계

1. 정수를 특정 진수의 문자열로 변환하는 과정을 설정.
2. 생성된 문자열을 영을 기준으로 나누어 배열을 확보.
3. 배열의 각 요소를 순회하며 유효한 숫자를 정수로 변환.
4. 소수 판정 함수를 통해 조건에 부합하는 소수의 개수를 누적하는 판정 과정 완성.

# 코드

    class Solution {
        public int solution(int n, int k) {
            String converted = Integer.toString(n, k);
            String[] parts = converted.split("0");
            int answer = 0;
            
            for (String part : parts) {
                if (part.equals("")) {
                    continue;
                }
                long num = Long.parseLong(part);
                if (isPrime(num)) {
                    answer++;
                }
            }
            
            return answer;
        }
        
        private boolean isPrime(long num) {
            if (num < 2) {
                return false;
            }
            for (long i = 2; i <= Math.sqrt(num); i++) {
                if (num % i == 0) {
                    return false;
                }
            }
            return true;
        }
    }
