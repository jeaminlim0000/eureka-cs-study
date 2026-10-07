# Week13 Java/Spring — Bean생명주기와 singleton

> 발표자: [최정현]　 발표일: [2026.10.06]　 주차 Issue: [#279](https://github.com/claudette-class/eureka-cs-study/issues/279)

## 1. 핵심 개념

**스프링 Bean**은 **제어의 역전(IoC)** 에 따라 스프링 컨테이너가 생성, 의존관계 설정, 그리고 생명주기 전반을 관리하는 객체입니다. 일반 자바 객체가 개발자의 `new` 키워드로 생성되고 참조가 끊기면 가비지 컬렉터(GC)에 의해 소멸되는 것과 달리, Bean은 컨테이너에 의해 체계적으로 관리됩니다. 

```text
스프링 컨테이너 생성 -> 빈 생성(인스턴스화) -> 의존관계 주입(DI) -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 컨테이너 종료 및 GC
```

| 구분 | 생성 및 관리 주체 | 특징 |
|---|---|---|
| **일반 Java 객체** | 개발자 직접 관리 | `new` 생성, 사용 후 참조 해제 시 GC 수거 |
| **스프링 Bean** | 스프링 컨테이너 (`ApplicationContext`) | 컨테이너가 생명주기 제어 및 의존성 주입 처리 |

스프링이 IoC를 통해 객체를 관리함으로써, 객체 간의 결합도가 낮아집니다. 만약 IoC를 사용하지 않고 직접 객체를 관리한다면 테스트 시 모킹(Mocking)이 불가능해지는 등의 여러 한계가 발생할 수 있습니다.

## 2. Spring 의존성 주입(DI) 사용 예시

스프링에서 의존성 주입(DI) 방식에는 크게 생성자 주입, Setter 주입, 필드 주입이 있습니다. 그중 가장 권장되는 방식은 객체의 불변성을 보장할 수 있는 **생성자 주입**입니다.

```java
@Service
public class OrderService {
    // 💡 생성자 주입 후 참조 대상(주소값)을 절대로 바꿀 수 없음 (불변성 보장)
    private final PaymentService paymentService;

    // 생성자가 1개만 있으면 @Autowired를 생략해도 스프링이 알아서 주입해 줍니다.
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

- `private final` 키워드를 사용해 불변성을 보장하며, 컴파일 시점에 누락 오류를 확인할 수 있습니다.
- 반면, 필드 주입(`@Autowired` 직접 명시)은 코드는 짧지만 테스트가 어렵고 순환 참조 감지가 늦어 권장되지 않습니다.
- 참고로, 직접 클래스 코드를 수정해 `@Component`를 붙일 수 없는 외부 라이브러리는 `@Configuration` 클래스 내부에서 `@Bean`을 통해 수동으로 컨테이너에 등록합니다.

## 3. 잘못 사용했을 때 생기는 문제

스프링 빈은 기본적으로 **싱글톤(Singleton)** 으로 관리됩니다. 따라서 인스턴스 변수에 상태(State)를 저장하여 관리하면 심각한 동시성 문제(Race Condition)가 발생할 수 있습니다.

```text
[안티 패턴: Stateful 싱글톤 빈의 동시 접근 문제]
A 사용자 10,000원 결제 요청 -> orderAmount = 10000 저장
(스레드 지연 중) B 사용자 5,000원 결제 요청 -> orderAmount = 5000 덮어쓰기
A 사용자 결제 처리 재개 -> 최종 금액 5,000원으로 결제되는 대형 사고 발생
```

```java
@Service
public class OrderService {
    // 🚨 문제점: 싱글톤 빈에 상태(Stateful)를 유지하는 멤버 변수 선언
    private int orderAmount;

    public void order(String user, int amount) {
        System.out.println("주문자: " + user + ", 금액: " + amount);
        this.orderAmount = amount; // 동시 요청 시 값이 덮어써짐
        
        try { Thread.sleep(100); } catch (Exception e) {} // 외부 API 연동 등 지연 가정
        
        System.out.println("조회된 최종 금액: " + this.orderAmount); 
    }
}
```

싱글톤 객체의 인스턴스 변수는 Heap 영역에 존재하여 모든 스레드가 공유합니다. 이를 예방하기 위해 싱글톤 빈은 반드시 인스턴스 변수가 없는 **무상태(Stateless) 구조**로 설계되어야 합니다. (단, `private final`로 주입받는 다른 서비스 객체나 Repository는 주입 대상도 Stateless이므로 필드로 가져도 안전합니다.)

## 4. 무상태(Stateless) 설계 시 주의점

동시성 사고와 보안 문제(권한 오염 등)를 방지하기 위해 빈을 무상태로 유지하는 3가지 대안 방식을 사용할 수 있습니다. 각 방식별로 사용 상황과 주의점을 잘 판단해야 합니다.

| 확인할 내용 | 판단 기준 |
|---|---|
| **파라미터 (Parameter)** | 메서드 간 데이터를 전달하는 일반적 비즈니스 로직. 데이터가 많으면 DTO 객체 활용 |
| **지역 변수 (Local Variable)** | 메서드 내부에서만 잠깐 사용하고 버리는 임시 데이터. 스레드 스택에만 존재하여 안전 |
| **`ThreadLocal`** | 한 번의 요청 동안 스레드 내부 어디서든 꺼내 써야 하는 공통 정보 (인증/추적 ID 등) |

`ThreadLocal`은 매우 편리하지만, 톰캣 등 스레드 풀(Thread Pool)을 사용하는 환경에서는 이전 요청의 정보가 남아 오염될 위험이 있습니다. 따라서 요청 처리가 완료되면(`Filter`나 `Interceptor`의 `finally` 구문 등) 반드시 `remove()`(또는 `clear()`) 메서드로 스레드 보관함을 비워주어야 합니다.

## 5. 종합 사례 — 주문 결제 방식 다형성과 빈 상태 관리

> "같은 타입의 결제 방식(카카오페이, 네이버페이, 신용카드 등) 인터페이스 구현 빈이 여러 개일 때, 런타임에 어떻게 중복 빈 에러를 피하고 다형성을 살릴 수 있나요?"

`PaymentService` 인터페이스를 구현한 여러 Bean(`KakaoPayService`, `CardPaymentService` 등)이 등록된 상태에서 단일 필드로 주입받으려 하면 스프링은 어떤 빈을 주입할지 몰라 에러를 발생시킵니다.

```text
[다형성을 활용한 동적 결제수단 선택 흐름]
사용자가 결제 수단 '카카오페이' 선택 -> OrderService 호출 -> 주입받은 Map에서 "kakaoPayService" 키 탐색 -> 해당 객체의 pay() 실행
```

| 상황 | 대응 방향 |
|---|---|
| 특정 구현체를 기본 결제 수단으로 사용하고자 할 때 | 기본 구현 클래스에 `@Primary` 어노테이션을 부여하여 우선순위를 지정 |
| 런타임에 동적으로 유연하게 구현체를 선택해야 할 때 | `Map<String, PaymentService>`을 통해 빈 이름을 Key로, 실제 객체를 Value로 모두 주입받아 선택하여 사용 (`if-else` 제거 가능) |
| 모든 결제 수단 목록 확인 및 검증이 필요할 때 | `List<PaymentService>` 타입으로 선언하여 구현된 모든 빈 리스트를 한 번에 주입받아 활용 |

**결론 및 요약:** 스프링 환경에서 다형성을 활용하여 `Map`이나 `List`로 여러 빈을 주입받으면 유연한 확장이 가능합니다. 단, 주입되는 모든 객체는 인스턴스 변수에 상태를 저장하지 않는 무상태(Stateless) 구조를 유지해야 하며, 요청별 상태 데이터는 파라미터나 지역 변수로 분리하여 스레드 독립성을 철저히 보장해야 합니다.

## 참고 자료

- [Spring Framework - Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Spring Framework - Bean Scopes (Singleton)](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
