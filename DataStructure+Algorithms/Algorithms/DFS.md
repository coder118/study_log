# DFS(depth-first search)

트리나 그래프를 탐색하는 기법 중 하나다.

시작 노드에서 자식의 노드들을 순서대로 탐색하며 깊이를 우선으로 탐색한다.

## 구현 방법

주로 반복문이나 재귀문을 통해 구현된다.
Stack을 사용할 수 있다.

## DFS 탐색 과정

기본 탐색 과정은 특정 정점에서 시작해 역추적(backtracking)하기 전에 각 분기를 따라 가능한 멀리 탐색하는 것이다. 


1. 현재 노드를 방문한 것으로 표시한다.
2. 방문한 표시가 되어 있지 않은 각각의 인접한 정점을 탐색한다.
3. 더 이상 방문하지 않은 정점이 없으면 이전 정점으로 역추적(backtracking) 한다.
4. 모든 정점을 방문할 때까지 프로세스를 반복한다.



![image](/DataStructure+Algorithms/DA_images/img1.daumcdn.png)



----

[dfs 공부 자료-1](https://velog.io/@falling_star3/2.-%EA%B9%8A%EC%9D%B4%EC%9A%B0%EC%84%A0%ED%83%90%EC%83%89DFS%EA%B3%BC-%EB%84%93%EC%9D%B4%EC%9A%B0%EC%84%A0%ED%83%90%EC%83%89BFS)


[dfs-stack과 재귀로 구현](https://olrlobt.tistory.com/35)