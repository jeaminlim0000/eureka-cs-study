# Sync · Async — 서희

> Week08 · 2026.09.04 · [Issue #9](https://github.com/jeaminlim0000/eureka-cs-study/issues/9)

Sync는 호출한 쪽이 결과를 받아야 다음 흐름을 결정하는 방식이고, Async는 완료 통지를 나중에 받거나 별도 경로로 처리합니다. 결제 승인은 성공·실패 결과가 즉시 필요하므로 동기로 처리하고, 알림은 메시지 큐·재시도·멱등성을 갖춘 비동기 후처리로 분리할 수 있습니다.

Blocking/Non-Blocking은 스레드의 대기 방식, Sync/Async는 결과를 받는 방식이므로 같은 개념이 아닙니다.
