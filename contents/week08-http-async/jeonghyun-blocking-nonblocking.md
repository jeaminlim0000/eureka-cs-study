# Blocking · Non-Blocking — 정현

> Week08 · 2026.09.04 · [Issue #9](https://github.com/jeaminlim0000/eureka-cs-study/issues/9)

Blocking은 I/O 결과가 올 때까지 호출 스레드가 기다리는 방식이고, Non-Blocking은 I/O가 준비되지 않았을 때 제어권을 즉시 돌려줍니다. 외부 API가 느리면 Tomcat 요청 스레드가 점유되어 스레드 풀이 고갈될 수 있습니다. Non-Blocking은 대기 스레드 수를 줄이지만 외부 API 자체의 응답 시간을 줄이지는 않습니다.
