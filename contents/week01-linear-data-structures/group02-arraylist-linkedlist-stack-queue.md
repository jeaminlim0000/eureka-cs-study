# 2조 발표 기록 — ArrayList · LinkedList · Stack · Queue

> Week01 · 2026.07.07 · [Issue #2](https://github.com/jeaminlim0000/eureka-cs-study/issues/2)

- 이서진: ArrayList, Queue
- 박수빈: LinkedList
- 최정현: Stack

ArrayList는 연속된 메모리 덕분에 인덱스 접근이 O(1)이고, 중간 삽입·삭제는 요소 이동 때문에 O(N)입니다. 배열이 꽉 차면 더 큰 배열을 만들고 기존 요소를 복사합니다.

LinkedList는 노드 연결 구조여서 위치를 알고 있는 삽입·삭제에 유리하지만, 위치 탐색은 O(N)입니다. Stack은 LIFO, Queue는 FIFO이며, Queue의 고정 배열 문제는 원형 큐로 완화할 수 있습니다.
