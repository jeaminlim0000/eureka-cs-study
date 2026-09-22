# Week08 Network HTTP와 비동기

> 일자: 2026.09.04 · 관련 Issue: [#9](https://github.com/jeaminlim0000/eureka-cs-study/issues/9)

## 발표·학습 구성

- 현빈: HTTP 기본
- 재민: HTTPS·TLS
- 정현: Blocking / Non-Blocking
- 서희: Sync / Async

## 함께 정리한 내용

- HTTP는 요청·응답 규칙이고, HTTPS는 TLS로 암호화·서버 인증·무결성을 제공합니다.
- Blocking은 I/O 완료를 기다리는 동안 스레드가 멈추는지, Non-Blocking은 제어권을 바로 돌려주는지에 관한 구분입니다.
- Sync/Async는 결과를 언제·어떤 방식으로 받는지에 관한 구분입니다.
- 결제 승인은 결과가 필요한 동기 작업이고, 알림은 메시지 큐·재시도·멱등성을 고려한 비동기 후처리로 분리할 수 있습니다.

## 종합 질문

외부 결제 API가 느려질 때는 HTTP 연결·TLS·요청 스레드 점유·동기 결제 호출을 분리해 살펴보고, 알림 같은 후처리는 비동기로 전환하되 실패 재시도와 중복 처리를 설계해야 합니다.
