# Scheduling · Deadlock · Mutex · Semaphore — 공동 기록

> Week05 · 2026.08.04 · [Issue #6](https://github.com/jeaminlim0000/eureka-cs-study/issues/6)

개인별 발표 배정 기록이 확인되지 않아 공동 발표 기록으로 남깁니다.

Scheduling은 CPU를 어떤 프로세스·스레드에 배분할지 정합니다. 선점형은 응답성이 좋지만 Context Switching 비용이 있습니다. Mutex는 한 스레드의 독점 접근, Semaphore는 정해진 수만큼의 동시 접근을 제어합니다.

Deadlock은 상호 배제, 점유와 대기, 비선점, 순환 대기가 모두 성립할 때 발생할 수 있습니다. 일관된 락 획득 순서, 타임아웃, 재시도는 완화 방법입니다.
