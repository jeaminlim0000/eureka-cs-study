# Week03 해시와 알고리즘

> 일자: 2026.07.21 · 관련 Issue: [#4](https://github.com/jeaminlim0000/eureka-cs-study/issues/4)

## 발표 기록 상태

노션에는 Hash, DFS/BFS, Sort, DP 질문 은행이 남아 있으나, 당일 발표자별 세부 배정·진행 기록은 비어 있습니다.

## 함께 정리한 내용

- Hash Table은 평균 O(1) 조회·삽입·삭제를 제공하지만, 충돌이 심하면 O(N)까지 저하될 수 있습니다.
- `HashMap` Key 객체는 `equals()`와 `hashCode()`를 함께 재정의하고 가능하면 불변으로 둡니다.
- BFS는 가까운 노드부터 탐색해 동일 가중치 그래프의 최단 거리에 적합하고, DFS는 깊이 탐색·사이클 검사에 적합합니다.
- DP는 중복되는 부분 문제의 결과를 저장해 재사용합니다. Top-down은 재귀·Memoization, Bottom-up은 반복문·테이블 채우기 방식입니다.
