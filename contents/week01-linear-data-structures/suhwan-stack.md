# Stack — 임수환

> Week01 · 2026.07.07 · [Issue #2](https://github.com/jeaminlim0000/eureka-cs-study/issues/2)

Stack은 마지막에 넣은 값이 먼저 나오는 LIFO 자료구조입니다. `push`, `pop`, `peek`은 모두 맨 위에서 수행하므로 O(1)입니다. 함수 호출, 웹페이지 뒤로 가기, 작업 취소에 활용할 수 있습니다.

고정 크기 배열로 구현하면 한도를 넘는 입력에서 overflow가 생길 수 있으므로 가변 자료구조나 크기 확장 정책을 고려합니다.
