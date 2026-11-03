# Week14 Spring MVC & JPA — 변경 감지(Dirty Checking)

> 발표자: [최정현]　 발표일: [2026.10.13]　 주차 Issue: [#[28]](링크)

## 1. 핵심 개념

**변경 감지(Dirty Checking)**는 엔티티의 상태 변경을 영속성 컨텍스트가 감지하여, `repository.save()`나 가상의 `em.update()` 같은 명시적 메서드 호출 없이 트랜잭션 커밋 또는 `flush()` 시점에 자동으로 `UPDATE` SQL을 실행하는 JPA의 핵심 메커니즘입니다.

```text
[변경 감지 동작 흐름]
1. 엔티티 조회 (1차 캐시에 '엔티티'와 '최초 스냅샷' 저장)
   ↓
2. 도메인 로직 수행 (자바 엔티티 객체 필드값 변경)
   ↓
3. 트랜잭션 커밋 / Flush 발생 (현재 엔티티 상태 vs 스냅샷 비교)
   ↓
4. Diff 발생 시 쓰기 지연 SQL 저장소에 UPDATE 쿼리 생성
   ↓
5. DB에 UPDATE 쿼리 일괄 전송 (커밋 시점에만 연결하여 실행)
```

1. **스냅샷 생성**: 엔티티를 영속성 컨텍스트(1차 캐시)에 최초로 올리는 시점의 원본 상태를 사본(스냅샷)으로 복사해 둡니다.
2. **상태 비교 (Diff)**: 커밋이나 플러시 이벤트가 발생하면, 현재 엔티티의 상태와 최초 스냅샷을 1:1로 비교합니다.
3. **쓰기 지연 SQL 수행**: 변경 사항이 감지되면 `UPDATE` SQL을 생성하여 쓰기 지연 저장소에 보관한 뒤, 커밋 시점에 한꺼번에 DB에 전달합니다. 변경이 일어날 때마다 매번 DB 커넥션을 붙잡고 있지 않아 연결 자원을 효율적으로 사용합니다.

> **💡 메모리 사용량과 최적화 고려사항**  
> 1차 캐시에 엔티티 인스턴스 외에 **스냅샷(복사본)**까지 함께 보관하므로 메모리 사용량이 약 2배로 증가합니다.  
> 따라서 단순 데이터 조회가 목적인 로직에는 `@Transactional(readOnly = true)`를 선언하면, 하이버네이트가 스냅샷 생성을 생략(Flush 모드를 MANUAL로 최적화)하여 메모리를 절약하고 CPU 성능을 높일 수 있습니다.

---

## 2. Java / Spring 사용 예시

`@Transactional` 범위 내에서 영속 엔티티를 조회하고 도메인 비즈니스 메서드로 값을 변경하면, 명시적인 저장 호출 없이도 변경 사항이 DB에 안전하게 반영됩니다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    @Transactional
    public void changeShippingAddress(Long orderId, String newAddress) {
        // 1. 엔티티 조회 (영속성 컨텍스트의 1차 캐시에 스냅샷과 함께 영속 상태로 저장)
        Order order = orderRepository.findById(orderId)
                .orElseThrow(() -> new IllegalArgumentException("주문을 찾을 수 없습니다."));

        // 2. 도메인 메서드를 통해 상태 변경 (자바 객체 값만 수정)
        // Setter를 남발하기보다 도메인의 의도가 드러나는 비즈니스 메서드 사용 권장
        order.changeAddress(newAddress);

        // 3. orderRepository.save(order)를 명시적으로 호출하지 않음!
        // 메서드 종료 시 @Transactional에 의해 커밋되면서 Dirty Checking 동작 -> UPDATE SQL 실행
    }
}
```

---

## 3. 자주 생기는 문제나 실수

### 문제 1: 새로 생성된 비영속 객체 수정 시 Diff 미동작
`new` 키워드로 직접 생성한 엔티티는 영속성 컨텍스트가 관리하지 않는 **비영속(New) 상태**입니다. 스냅샷 자체가 존재하지 않으므로 변경 감지가 동작하지 않습니다.

```java
@Transactional
public void updateOrder(OrderDto dto) {
    Order order = new Order(); // 🚨 영속성 컨텍스트가 관리하지 않는 비영속 객체
    order.setId(dto.getId());
    order.setAddress(dto.getAddress());
    
    // 이 상태로 트랜잭션이 끝나도 Dirty Checking은 전혀 동작하지 않고 아무 일도 일어나지 않음!
}
```

### 문제 2: `merge()` 사용 시 발생하는 `null` 덮어쓰기 사고
비영속/준영속 객체를 영속 상태로 바꾸기 위해 `merge()`를 사용할 수 있지만, **실무에서는 매우 위험하여 거의 사용하지 않습니다.** `merge()`는 넘어온 객체의 모든 속성으로 기존 엔티티를 통째로 교체하기 때문에, 미입력된 `null` 값까지 DB에 덮어써지는 대형 사고가 발생합니다.

- **상황:** 화면에서 회원의 이름(`name`)만 수정 요청하여 나머지 필드는 DTO에 넘어오지 않은 경우
```java
// 화면에서 name만 넘어오므로 나머지 필드는 채우지 않음
MemberUpdateDto dto = new MemberUpdateDto();
dto.setId(1L);
dto.setName("홍길동_NEW"); // email, phone, address는 넘어오지 않아 null 상태
```

| 컬럼 | DB 기존 데이터 | 수정 요청 DTO | `merge()` 실행 후 DB 데이터 |
|---|---|---|---|
| `id` | `1L` | `1L` | `1L` |
| `name` | "홍길동" | "홍길동_NEW" | "홍길동_NEW" (수정됨) |
| `email` | "h@t.com" | `null` | **`null` (데이터 유실 🚨)** |
| `phone` | "010-1234" | `null` | **`null` (데이터 유실 🚨)** |
| `address` | "서울시 강남구" | `null` | **`null` (데이터 유실 🚨)** |

> **해결책:** DTO로 비영속 객체를 만들어 `merge()`하지 말고, 식별자(`id`)로 DB에서 엔티티를 직접 조회하여 **영속 상태로 만든 뒤 필요한 필드만 변경 감지로 수정**해야 합니다.

---

## 4. 실무 설계 및 운영 시 주의점 (전체 UPDATE vs `@DynamicUpdate`)

JPA는 기본적으로 특정 필드만 수정되더라도 **모든 컬럼을 포함한 UPDATE 쿼리**를 생성합니다.

| 구분 | 기본 UPDATE (모든 컬럼 반영) | `@DynamicUpdate` (변경된 컬럼만 반영) |
|---|---|---|
| **장점** | 애플리케이션 로딩 시점에 쿼리 형태가 미리 고정되어 파싱되므로, DB 및 애플리케이션의 **PreparedStatement 재사용성/캐싱 효율 극대화** | 실제 수정된 컬럼만 전송하므로 네트워크 데이터 전송량 감소 |
| **단점** | 변경되지 않은 컬럼 데이터도 함께 전송됨 | 엔티티 수정 시마다 런타임에 **SQL을 동적으로 새로 생성하고 파싱하는 오버헤드** 발생 |
| **선택 기준** | **기본 권장 사항** (대부분의 실무 도메인에 적합) | 한 테이블에 컬럼이 수십 개(약 30개 이상)로 매우 많거나, 대용량 텍스트/바이너리 컬럼이 자주 수정되는 테이블에 한해 제한적 적용 |

---

## 5. 종합 사례 — 주문 정보 수정 흐름과 대량(벌크) 수정 최적화

> "주문 서비스에서 사용자가 기존 주문 정보를 수정할 때 Dirty Checking이 어떻게 반영되며, 대량 수정 시 어떤 성능 문제와 주의점이 발생하나요?"

### 1) 단건 주문 정보 수정 흐름
1. `Service` 계층에서 `orderRepository.findById(orderId)`를 호출해 엔티티를 조회하면 영속성 컨텍스트에 1차 캐시와 최초 스냅샷이 생성됩니다.
2. 도메인 메서드(`order.changeAddress(...)`)를 호출하여 메모리 상의 엔티티 필드값을 변경합니다.
3. 서비스 메서드가 종료되는 시점에 트랜잭션 커밋이 실행되며 `flush()`가 발생합니다.
4. 영속성 컨텍스트가 엔티티와 스냅샷의 차이(Diff)를 감지하여 `UPDATE` SQL을 생성하고, DB에 전송하여 수정사항을 최종 반영합니다.

### 2) 대량 수정 시 발생하는 성능 문제와 해결 (N번 쿼리 방지)
- **문제점:** 루프를 돌며 수천~수만 건의 엔티티를 조회하고 각각 필드를 수정하면, 건수만큼 개별 `UPDATE` 쿼리가 전송되어 네트워크 I/O(DB 접속 시간)가 급증하고 시스템 병목이 발생합니다.
- **해결책:** 대량 수정 작업에는 Dirty Checking 대신 **JPQL 벌크 연산(`@Modifying`)**을 사용하여 단 1번의 쿼리로 다건을 일괄 수정합니다.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // 벌크 연산 시 1차 캐시와의 데이터 불일치를 방지하기 위해 clearAutomatically 설정 필수
    @Modifying(clearAutomatically = true)
    @Query("UPDATE Order o SET o.status = :status WHERE o.createdAt < :date")
    int bulkUpdateOrderStatus(@Param("status") OrderStatus status, @Param("date") LocalDateTime date);
}
```

> **🚨 벌크 연산 시 필수 주의점 (영속성 컨텍스트 우회 문제)**  
> 벌크 연산은 1차 캐시를 거치지 않고 DB에 직접 쿼리를 날립니다. 따라서 메모리 상의 영속성 컨텍스트와 DB 데이터 사이에 불일치가 생길 수 있으므로, 반드시 **`clearAutomatically = true`** 옵션을 부여하여 쿼리 실행 직후 1차 캐시를 비워주어야 후속 조회 시 꼬임 현상을 방지할 수 있습니다.

---

## 참고 자료

- [Hibernate ORM Documentation - Persistence Contexts](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#pc)
- [Spring Data JPA - Modifying Queries](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.modifying-queries)
