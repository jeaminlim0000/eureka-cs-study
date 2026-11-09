# 면접 1주차 · 서희 (@superparkgit)

- 학습 주제: 인터페이스와 다형성 / IoC와 생성자 주입 / Spring 빈 등록과 주입 대상 선택
- 정리일: 2026.11.09
- 스터디일: 2026.11.10
- 기존 학습: [13주차 Spring Core](https://github.com/jeaminlim0000/eureka-cs-study/issues/27)

## 1. 인터페이스와 다형성

### 핵심 개념

인터페이스는 객체가 제공할 동작의 계약을 정의합니다. 호출하는 쪽은 그 계약을 사용하고, 실행 시에는 실제 객체가 구현한 인스턴스 메서드가 동작할 수 있습니다.

다형성은 같은 타입으로 여러 구현 객체를 다루면서 실제 객체에 따라 다른 동작을 수행하는 성질입니다. 인터페이스를 선언하는 것만으로 구현 선택과 객체 생성까지 해결되는 것은 아닙니다. 어떤 구현을 사용할지는 호출부 밖의 조립 코드나 컨테이너에서 결정할 수 있습니다.

인터페이스와 추상 클래스는 변경되는 부분과 공유할 구현을 기준으로 비교합니다. 인터페이스는 역할과 계약을 표현하는 데 사용하고, 추상 클래스는 관련 클래스 사이의 상태나 공통 구현을 함께 설계할 때 검토합니다. 인터페이스에도 기본 메서드 등이 있으므로 구현 유무만으로 둘을 구분하지 않습니다.

### 동작 원리와 코드

결제 수단이 달라져도 주문 서비스가 같은 계약으로 요청하도록 구성한 학습용 코드입니다.

```java
interface PaymentGateway {
    String pay(long amount);
}

class CardGateway implements PaymentGateway {
    @Override
    public String pay(long amount) {
        return "카드 결제: " + amount;
    }
}

class PointGateway implements PaymentGateway {
    @Override
    public String pay(long amount) {
        return "포인트 결제: " + amount;
    }
}

class OrderService {
    private final PaymentGateway paymentGateway;

    OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    String placeOrder(long amount) {
        return paymentGateway.pay(amount);
    }
}
```

`OrderService`의 `placeOrder()`는 주입된 객체의 `pay()`를 호출합니다. `CardGateway`를 전달하면 카드 구현이, `PointGateway`를 전달하면 포인트 구현이 실행됩니다.

이 구조에서 줄어드는 결합은 주문 서비스가 특정 결제 구현을 선택하고 생성하는 책임입니다. 결제 인터페이스의 계약이 바뀌면 서비스에도 영향이 생길 수 있으므로 결합이 모두 사라진다고 설명하지 않습니다.

### 적용을 판단할 기준

- 결제 수단처럼 실제로 교체되거나 확장되는 경계가 있는지 확인합니다.
- 외부 통신을 테스트용 구현으로 대체할 필요가 있는지 확인합니다.
- 구현이 하나뿐이고 변경 경계도 없다면 인터페이스를 별도로 관리하는 비용을 함께 비교합니다.
- 메서드 이름만 맞추는 것이 아니라 입력, 결과, 실패 처리의 계약도 일관되게 설계합니다.

### 참고 자료

- [Java 인터페이스와 상속](https://docs.oracle.com/javase/tutorial/java/IandI/index.html)
- [Java 다형성](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html)

## 2. IoC와 생성자 주입

### 핵심 개념

IoC는 객체 생성과 연결 같은 제어를 외부 구성 요소에 맡기는 관점입니다. DI는 객체가 필요한 의존성을 외부에서 전달받는 방식입니다. Spring IoC 컨테이너는 빈을 생성하고 DI를 통해 의존 관계를 구성합니다.

DI와 의존성 역전 원칙(DIP)은 구분합니다. DI는 의존 객체를 전달하는 방식이고, DIP는 상위 정책과 세부 구현이 추상화에 의존하도록 설계하는 원칙입니다. 구체 클래스를 생성자로 전달받아도 DI이며, 인터페이스를 두었다는 사실만으로 DIP를 모두 만족한다고 단정할 수는 없습니다.

### 일반 Java에서 객체를 조립하는 흐름

```java
PaymentGateway gateway = new CardGateway();
OrderService orderService = new OrderService(gateway);
String result = orderService.placeOrder(10000L);
```

객체를 조립하는 코드가 카드 구현을 만들고 서비스에 전달합니다. 서비스 내부에서 카드 구현을 직접 생성하지 않으므로 생성과 사용을 나눌 수 있습니다. 이 예시는 Spring 없이도 생성자 주입이 가능하다는 점을 보여줍니다.

### 주입 방식 비교

| 방식 | 의존성을 받는 지점 | 확인할 특징 |
| --- | --- | --- |
| 생성자 주입 | 생성자 매개변수 | 필수 의존성을 드러내며 `final` 필드를 사용할 수 있음 |
| 세터 주입 | 생성 후 설정 메서드 | 선택적 의존성이나 재설정 요구에 활용할 수 있음 |
| 필드 주입 | 컨테이너가 필드에 주입 | 일반 Java 코드에서 직접 생성하면 별도의 주입 작업이 필요함 |

`final`은 참조 재할당을 제한합니다. 결제 객체 내부의 상태까지 불변이 되거나 동시 요청이 자동으로 안전해지는 것은 아닙니다. 일반 Java 코드에서 생성자에 `null`을 전달하는 것도 가능하므로 필요한 검증은 따로 고려합니다.

### Spring과 Lombok의 역할

- Spring은 등록 정보에 따라 빈을 생성하면서 생성자에 필요한 의존 빈을 전달합니다.
- 생성자가 하나인 빈은 생성자에 `@Autowired`가 없어도 그 생성자를 사용할 수 있습니다.
- Lombok의 `@RequiredArgsConstructor`는 초기화되지 않은 `final` 필드 등 필요한 필드에 대한 생성자를 생성합니다. 빈을 등록하거나 주입할 구현체를 선택하는 기능은 아닙니다.
- 생성자로 서로를 요구하는 두 빈은 순환 의존성으로 생성이 막힐 수 있습니다. 먼저 책임과 의존 방향을 검토합니다.

### 단위 테스트와 통합 검증의 경계

단위 테스트에서는 `PaymentGateway`의 테스트 대역을 직접 전달할 수 있습니다. 고정 결과를 반환하는 경우와 지정한 예외를 던지는 경우를 나누어, 서비스가 전달한 금액과 실패 처리 결과를 확인하는 방법을 사용할 수 있습니다.

이 검증만으로 Spring의 빈 등록, 실제 결제 서버와의 통신, 직렬화와 인증 설정이 정상이라는 사실까지 확인할 수는 없습니다. 빈 연결은 컨텍스트를 사용하는 검증으로, 실제 통신 계약은 별도의 통합 검증으로 확인해야 합니다.

### 참고 자료

- [Spring 의존성 주입](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Spring 생성자 자동 주입](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [Lombok 생성자 생성](https://projectlombok.org/features/constructor)

## 3. Spring 빈 등록과 주입 대상 선택

### 핵심 개념과 등록 방식

Spring 빈은 컨테이너가 관리하는 객체입니다. 같은 클래스로 직접 만든 객체와 빈은 생성과 관리 경로가 다를 수 있습니다. `new`로 만든 객체에 빈과 같은 주입이나 프록시 동작이 자동 적용된다고 가정하지 않습니다.

| 등록 방식 | 용도 |
| --- | --- |
| `@Component`와 컴포넌트 스캔 | 스캔 범위 안의 클래스를 빈으로 등록 |
| `@Service`, `@Repository` | 컴포넌트 역할과 계층의 의도를 표현하는 특수화 어노테이션 사용 |
| `@Configuration`의 `@Bean` 메서드 | 직접 구성하거나 외부 라이브러리 객체를 등록할 때 활용 |

### 구성 코드 예시

앞의 Java 클래스들을 Spring 빈으로 연결하는 학습용 구성입니다.

```java
@Configuration
class PaymentConfig {
    @Bean
    PaymentGateway cardGateway() {
        return new CardGateway();
    }

    @Bean
    PaymentGateway pointGateway() {
        return new PointGateway();
    }

    @Bean
    OrderService orderService(
            @Qualifier("cardGateway") PaymentGateway paymentGateway
    ) {
        return new OrderService(paymentGateway);
    }
}
```

위 코드는 Spring의 `Bean`, `Configuration`, `Qualifier` 어노테이션을 사용하며, 설정 클래스가 컨텍스트에 등록되어 있다는 전제입니다. 구성 메서드의 매개변수에서 카드 빈을 선택하고, 선택된 객체를 서비스 생성자에 전달합니다.

### 후보 선택과 요청별 선택의 차이

- `@Qualifier`는 주입 지점에서 타입에 맞는 후보를 더 좁힙니다.
- `@Primary`는 단일 값을 주입할 후보가 여러 개일 때 우선할 빈을 지정합니다.
- 여러 구현을 함께 사용해야 한다면 `List<PaymentGateway>`나 `Map<String, PaymentGateway>` 형태의 주입을 검토할 수 있습니다. Map의 키는 빈 이름이며, 사용자 입력과 직접 연결할지는 별도 정책으로 결정합니다.
- 빈 하나를 기본으로 주입하는 결정과 요청마다 결제 수단을 선택하는 결정은 다릅니다. 요청별 선택은 서비스나 별도 선택 객체에서 검증된 기준으로 처리할 수 있습니다.

### 시작 오류를 확인하는 순서

1. 필요한 빈이 없는지, 후보가 여러 개여서 선택할 수 없는지 오류를 구분합니다.
2. 필요한 타입과 실제 등록된 빈, 컴포넌트 스캔 범위와 활성 설정을 확인합니다.
3. 한 구현을 지정해야 하는지, 기본 구현이 필요한지, 여러 구현을 함께 써야 하는지 요구사항을 확인합니다.
4. 실제 주입 지점에 선택 조건이 적용됐는지 확인하고 컨텍스트 시작과 선택된 구현을 검증합니다.

Lombok을 쓰는 코드에서는 필드의 `@Qualifier`가 생성자 매개변수로 복사되는지 설정과 생성 결과를 확인합니다. 필요한 경우 생성자를 직접 작성해 주입 지점을 명시할 수 있습니다.

### 싱글턴 빈과 요청별 상태

기본 싱글턴 빈은 한 컨테이너의 해당 빈 정의에 대해 하나의 인스턴스를 공유합니다. 결제 금액이나 사용자 ID처럼 요청마다 달라지는 값을 공유 필드에 저장하면 서로 덮어쓸 수 있습니다. 이런 값은 메서드 매개변수와 지역 변수로 다루되, 참조가 가리키는 객체가 공유되는지도 확인합니다.

### 참고 자료

- [Spring 컴포넌트 스캔](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)
- [Spring Java 기반 빈 구성](https://docs.spring.io/spring-framework/reference/core/beans/java/bean-annotation.html)
- [Spring 주입 후보 선택](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)
- [Spring 빈 범위](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)

## 확인 범위

공통 주제 세 개를 정리한 개인 개념 노트입니다. 코드에는 학습용 결제 객체를 사용했으며, 실제 프로젝트의 구현·테스트 경험을 나타내지는 않습니다. 이 문서의 코드와 Spring 설정은 실행 검증하지 않았습니다. 면접 질문과 이후 답변·비교 기록은 Discussion에 남깁니다.
