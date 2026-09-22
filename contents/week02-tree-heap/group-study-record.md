# 트리 · BST · Heap · Binary Search — 공동 기록

> Week02 · 2026.07.14 · [Issue #3](https://github.com/jeaminlim0000/eureka-cs-study/issues/3)

당일 발표자별 배정 기록은 남아 있지 않아 공동 발표 기록으로 보관합니다.

- BST는 균형 상태에서 탐색·삽입·삭제 O(log N)이지만 편향되면 O(N)입니다.
- Heap은 우선순위가 가장 높은 값을 루트에서 O(1)에 조회하고, 삽입·삭제는 O(log N)입니다.
- `PriorityQueue<>(array)`의 heapify는 O(N)이며, 하나씩 추가하면 O(N log N)입니다.
- B-Tree는 한 노드에 여러 키를 담아 디스크 I/O를 줄이므로 DB 인덱스에 적합합니다.
