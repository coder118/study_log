# 문제 정의


우선 Arrays.sort 함수를 이용해 citations 배열을 정렬하게 되면 특정 인덱스 i부터 citations배열의 길이까지 만큼의 논문이 citations[i]번 만큼 인용되었다고 할 수 있다
citations[i]에서 i값을 증가시키고 논문의 수를 감소시키면서 비교 했을 때 인용 횟수가 논문의 수 이상이 되었을 때의 논문의 수가 H-Index가 된다

# 코드

    import java.util.*;

    class Solution {
        public int solution(int[] citations) {
            int answer = 0;
            
            Arrays.sort(citations);
            
            for(int i = 0; i < citations.length; i++) {
                int h = citations.length - i; // 인용된 논문의 수
                
                if(citations[i] >= h) {
                    answer = h;
                    break;
                }
            }
            
            return answer;
        }
    }