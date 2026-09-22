# Transaction — 재민

> Week10 · 2026.09.15 · [Issue #11](https://github.com/jeaminlim0000/eureka-cs-study/issues/11)

Transaction은 ACID를 통해 여러 DB 작업의 정합성을 보장합니다. 주문·결제 처리에서 DB 변경은 하나의 트랜잭션으로 묶되, 느린 외부 API 호출을 긴 트랜잭션에 포함하면 커넥션과 락을 오래 점유할 수 있으므로 분리 전략을 고려합니다.
