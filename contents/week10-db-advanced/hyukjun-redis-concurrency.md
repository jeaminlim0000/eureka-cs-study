# Redis 동시성 제어 — 혁준

> Week10 · 2026.09.15 · [Issue #11](https://github.com/jeaminlim0000/eureka-cs-study/issues/11)

재고 차감처럼 동시에 같은 자원을 변경하는 상황에서는 원자적 DB 업데이트, DB 락, Redis 분산 락을 상황에 맞게 검토합니다. Redis 분산 락은 만료 시간과 소유자 검증이 필요하며, 락을 썼다는 사실만으로 DB 정합성이 자동 보장되지는 않습니다.
