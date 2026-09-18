# 문제 정의


### 제약 조건 분석

- n의 크기가 12 이하로 매우 작다. 
- 백트래킹 탐색을 이용해 해결 가능

---

### 패턴 단순화 및 추상화

- 체스판의 한 행씩 내려가면서 퀸을 놓을 수 있는 열을 탐색
- 동일한 열이나 대각선 상에 이미 다른 퀸이 존재하는지 확인
- 대각선 확인 시 두 점의 행 차이와 열 차이의 절대값이 같으면 같은 대각선 상에 위치함
- 퀸 배치가 불가능한 경로면 이전 단계로 돌아가 다른 열을 시도

---

### 경계 조건

- 첫 번째 행부터 시작하여 마지막 행까지 퀸 배치를 완료하면 성공 조건 달성
- 퀸 n개를 모두 성공적으로 배치한 순간마다 방법의 수를 1씩 증가
- 배열 인덱스는 0부터 n 미만까지의 범위 내에서 관리

---

### 최적의 알고리즘

1. 각 행에 배치된 퀸의 열 위치를 기록할 1차원 배열을 생성
2. 현재 행에 퀸을 놓을 열을 0부터 n 미만까지 순회하며 배치를 시도.
3. 이전 행들에 놓인 퀸들과 열이 같거나 대각선 위치에 있는지 검증.
4. 공격받지 않는 안전한 위치라면 다음 행으로 이동하여 재귀적으로 퀸을 배치.
5. 마지막 행까지 퀸을 모두 배치하는 데 성공하면 배치 가능 카운트를 1 올리고 이전 단계로 돌아간다.

---

# 코드

    class Solution {
        private int count = 0;
        private int[] board;

        public int solution(int n) {
            board = new int[n];
            backtrack(0, n);
            return count;
        }

        private void backtrack(int row, int n) {
            if (row == n) {
                count++;
                return;
            }

            for (int col = 0; col < n; col++) {
                board[row] = col;
                if (isValid(row)) {
                    backtrack(row + 1, n);
                }
            }
        }

        private boolean isValid(int row) {
            for (int i = 0; i < row; i++) {
                if (board[i] == board[row]) {
                    return false;
                }
                if (Math.abs(row - i) == Math.abs(board[row] - board[i])) {
                    return false;
                }
            }
            return true;
        }
    }
