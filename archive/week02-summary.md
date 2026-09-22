# Week02 트리와 힙

> 일자: 2026.07.14 · 관련 Issue: [#3](https://github.com/jeaminlim0000/eureka-cs-study/issues/3)

## 발표 기록 상태

노션에는 Tree, BST, Heap, Binary Search 질문 은행은 남아 있으나, 당일 발표자별 세부 배정·진행 기록은 비어 있습니다.

## 함께 정리한 내용

- BST는 균형 상태에서 탐색·삽입·삭제가 O(log N)이지만 편향되면 O(N)이 될 수 있습니다.
- Heap은 최솟값 또는 최댓값을 빠르게 꺼내는 완전 이진 트리입니다. 삽입·삭제 O(log N), 루트 조회 O(1)입니다.
- `PriorityQueue<>(array)`의 heapify는 O(N)이고, 하나씩 `add`하면 O(N log N)입니다.
- B-Tree는 한 노드에 여러 키를 담아 디스크 I/O를 줄이기 때문에 DB 인덱스에 적합합니다.
