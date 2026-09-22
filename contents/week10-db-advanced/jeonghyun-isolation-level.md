# Isolation Level — 정현

> Week10 · 2026.09.15 · [Issue #11](https://github.com/jeaminlim0000/eureka-cs-study/issues/11)

Isolation Level은 동시 트랜잭션에서 Dirty Read, Non-repeatable Read, Phantom Read를 어느 수준까지 막을지 정합니다. 격리 수준을 올릴수록 정합성은 좋아질 수 있지만 락 경합과 처리량 저하를 함께 고려해야 합니다.
