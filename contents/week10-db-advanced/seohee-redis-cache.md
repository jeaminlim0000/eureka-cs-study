# Redis Cache — 서희

> Week10 · 2026.09.15 · [Issue #11](https://github.com/jeaminlim0000/eureka-cs-study/issues/11)

Cache-Aside는 먼저 Redis를 조회하고 없으면 DB에서 읽은 뒤 캐시에 저장하는 방식입니다. TTL, 데이터 변경 시 무효화, Cache Stampede를 함께 설계해야 합니다. 캐시는 원본 DB와 순간적으로 달라질 수 있으므로 정합성 요구가 높은 데이터는 적용 범위를 신중하게 정합니다.
