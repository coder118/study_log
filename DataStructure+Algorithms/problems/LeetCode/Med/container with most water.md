# 문제 정의

[링크](https://leetcode.com/problems/container-with-most-water/?envType=problem-list-v2&envId=array)

### 제약 조건 분석

* 용기를 기울여서 물을 담는 방식은 허용되지 않음.
* 용기의 높이는 선택한 두 선 중 더 낮은 선의 높이로 결정.
* 용기의 너비는 선택한 두 선의 위치 차이인 인덱스 간격으로 결정.

---

### 패턴 단순화 및 추상화

* 용기의 넓이는 두 선 사이의 거리와 두 선 중 낮은 높이의 곱으로 표현.
* 배열의 양 끝에 두 개의 포인터를 위치시킨 후 탐색을 시작하는 구조.
* 포인터를 이동할 때마다 너비가 줄어들므로, 높이가 낮은 쪽을 이동시켜야 더 큰 넓이를 찾을 가능성이 존재.
* 매 단계마다 계산된 넓이 중 최댓값을 지속적으로 갱신하여 관리.

---

### 경계 조건

* 입력 배열의 길이가 최소값인 2인 경우를 확인.
* 선의 높이가 0으로 설정되어 담을 수 있는 물의 양이 0인 경우를 확인.
* 모든 선의 높이가 동일하여 너비가 가장 넓은 양 끝을 선택하는 것이 최선인 상황을 확인.
* 양 끝선보다 내부의 선이 매우 높아 포인터 이동 후 넓이가 더 증가하는 상황을 확인.

---

### 최적의 알고리즘

1. 왼쪽 끝과 오른쪽 끝을 가리키는 두 개의 포인터 설정.
2. 현재 두 포인터 위치의 선 높이 중 작은 값과 두 포인터 사이의 거리를 곱하여 넓이 산출.
3. 산출된 넓이와 기존 최댓값을 비교하여 최댓값을 지속적으로 갱신.
4. 두 선의 높이 중 더 낮은 높이를 가진 쪽의 포인터를 안쪽으로 한 칸 이동.
5. 두 포인터가 서로 만날 때까지 위의 계산 및 이동 과정을 반복 수행하여 최종 최댓값 확인.

# 코드

    class Solution {
        public int maxArea(int[] height) {
            int left = 0;
            int right = height.length - 1;
            int maxWater = 0;

            while (left < right) {
                int currentWidth = right - left;
                int currentHeight = Math.min(height[left], height[right]);
                int currentArea = currentWidth * currentHeight;

                maxWater = Math.max(maxWater, currentArea);

                if (height[left] < height[right]) {
                    left++;
                } else {
                    right--;
                }
            }

            return maxWater;
        }
    }