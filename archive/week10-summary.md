# Week10 DB 심화

> 일자: 2026.09.15 · 관련 Issue: [#11](https://github.com/jeaminlim0000/eureka-cs-study/issues/11)

## 발표·학습 구성

- 현빈: Index
- 재민: Transaction
- 정현: Isolation Level
- 서희: Redis Cache
- 혁준: Redis 동시성 제어

## 함께 정리한 내용

- Index는 조회를 빠르게 하지만 삽입·수정·삭제 때 인덱스도 갱신해야 하므로 쓰기 비용이 증가합니다.
- Transaction은 ACID를 통해 여러 DB 작업의 정합성을 보장합니다. 외부 API 호출을 긴 트랜잭션에 넣으면 커넥션과 락을 오래 점유할 수 있습니다.
- Isolation Level은 Dirty Read, Non-repeatable Read, Phantom Read와 정합성·동시성의 트레이드오프를 조절합니다.
- Redis Cache는 Cache-Aside, TTL, 캐시 불일치, Cache Stampede를 함께 고려해야 합니다.
- 재고 동시성은 DB의 원자적 업데이트·낙관/비관 락·Redis 분산 락을 상황에 맞게 검토합니다.

## 기록 상태

노션에는 Transaction과 Isolation Level 상세 기록만 남아 있으며, Index·Redis Cache·Redis 동시성의 상세 발표 본문은 누락되어 있습니다.
