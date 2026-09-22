# Queue — 박서희

> Week01 · 2026.07.07 · [Issue #2](https://github.com/jeaminlim0000/eureka-cs-study/issues/2)

Queue는 먼저 들어온 데이터가 먼저 나가는 FIFO 자료구조입니다. 삽입은 Rear에서, 삭제는 Front에서 수행하므로 각각 O(1)입니다. 원하는 값을 찾으려면 앞에서부터 확인하므로 탐색은 O(N)입니다.

고정 크기 배열 Queue는 앞쪽에 빈 공간이 생겨도 Rear가 배열 끝에 도달하면 삽입하지 못하는 가짜 포화 상태가 생길 수 있습니다. Front와 Rear를 순환시키는 원형 큐로 빈 공간을 재사용할 수 있습니다.
