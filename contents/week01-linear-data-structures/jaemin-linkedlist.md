# LinkedList — 임재민

> Week01 · 2026.07.07 · [Issue #2](https://github.com/jeaminlim0000/eureka-cs-study/issues/2)

LinkedList는 각 노드가 데이터와 다음 노드의 참조를 가지고 연결되는 자료구조입니다. 연속된 메모리에 저장되지 않으므로 인덱스 접근은 O(N)입니다. 위치를 이미 알고 있다면 연결만 바꿔 삽입·삭제할 수 있어 O(1)이지만, 일반적으로는 위치 탐색 비용이 먼저 듭니다.

실무에서는 순차 조회와 캐시 지역성 때문에 기본 컬렉션으로 `ArrayList`를 더 자주 선택합니다.
