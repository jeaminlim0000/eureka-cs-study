# Week01 선형 자료구조

> 일자: 2026.07.07 · 관련 Issue: [#2](https://github.com/jeaminlim0000/eureka-cs-study/issues/2)

## 발표·학습 구성

1조는 Array, 임재민의 LinkedList, 임수환의 Stack, 박서희의 Queue를 다뤘습니다. 2조는 이서진의 ArrayList·Queue, 박수빈의 LinkedList, 최정현의 Stack을 다뤘습니다.

## 함께 정리한 내용

- Array/ArrayList는 연속된 메모리에 저장되어 인덱스 접근이 O(1)이고, 중간 삽입·삭제는 요소 이동 때문에 O(N)입니다.
- LinkedList는 노드가 포인터로 연결됩니다. 위치를 알고 있으면 삽입·삭제가 O(1)이지만, 위치를 찾는 데 O(N)이 들 수 있습니다.
- Stack은 LIFO, Queue는 FIFO입니다. 고정 배열 Queue의 가짜 포화 상태는 원형 큐로 해결할 수 있습니다.

## 질문 기록

고정 배열 Queue에서 앞 공간이 비어도 Rear가 끝에 도달하면 삽입할 수 없는 문제가 생깁니다. 전체 이동 대신 원형 인덱스를 사용하면 O(1)로 빈 공간을 재사용할 수 있습니다.
