# Week13 · 의존성 주입(DI)

- 작성자 · 서희 (@superparkgit)
- 학습 일정 · 2026.10.06, 주차 이슈 기준
- 기록 보완일 · 2026.11.07
- 주차 이슈 · [Spring Core #27](https://github.com/jeaminlim0000/eureka-cs-study/issues/27)

## 핵심 개념

의존성은 한 객체가 자신의 일을 처리하기 위해 사용하는 다른 객체입니다. DI는 필요한 객체를 외부에서 전달하는 방식입니다. 호출하는 코드에서 생성자로 전달할 수도 있고, Spring에서는 컨테이너가 빈을 생성하면서 의존성을 연결할 수 있습니다.

IoC는 객체의 생성과 연결에 대한 제어를 외부 구성 요소에 맡기는 관점입니다. Spring IoC 컨테이너는 빈의 등록 정보를 바탕으로 객체를 만들고, DI를 통해 의존 관계를 구성합니다.

## 생성자 주입의 동작

1. 컨테이너가 서비스와 의존 객체의 빈 등록 정보를 읽습니다.
2. 서비스를 생성할 때 생성자 매개변수에 필요한 타입을 확인합니다.
3. 해당 타입에 맞는 빈을 찾아 생성자에 전달합니다.
4. 생성된 서비스는 전달받은 객체를 사용해 자신의 일을 처리합니다.

생성자가 하나인 빈은 `@Autowired`를 생략할 수 있습니다. Lombok의 `@RequiredArgsConstructor`를 쓰면 초기화되지 않은 `final` 필드 등에 대한 생성자가 만들어지고, Spring은 그 생성자를 사용할 수 있습니다. 생성자 생성과 의존 객체 선택·주입은 각각 다른 단계입니다.

## 주입 방식 비교

| 방식 | 의존성 전달 시점과 특징 |
| --- | --- |
| 생성자 주입 | 생성 시 필수 의존성을 드러내며 `final` 필드를 사용할 수 있음 |
| 세터 주입 | 생성 후 메서드로 설정하며, 선택적인 의존성이나 재설정 요구에 활용 가능 |
| 필드 주입 | 컨테이너가 필드에 주입하며, 일반 Java 코드에서 직접 객체를 구성하는 테스트에는 추가 작업이 필요 |

`final`은 참조의 재할당을 제한합니다. 참조하는 객체 내부의 상태가 모두 불변이 된다는 뜻은 아닙니다. 일반 Java 코드에서 생성자에 `null`을 전달하는 것도 가능하므로 필요한 입력 검증은 따로 고려합니다.

## Java와 Spring 예시

아래는 알림 발송 구현을 서비스에 전달하는 학습용 코드입니다.

```java
public interface MessageSender {
    String send(String message);
}
```

```java
import org.springframework.stereotype.Component;

@Component
public class EmailMessageSender implements MessageSender {
    @Override
    public String send(String message) {
        return "이메일 예시 · " + message;
    }
}
```

```java
import org.springframework.stereotype.Service;

@Service
public class NotificationService {
    private final MessageSender messageSender;

    public NotificationService(MessageSender messageSender) {
        this.messageSender = messageSender;
    }

    public String sendNotification(String message) {
        return messageSender.send(message);
    }
}
```

컴포넌트 스캔 범위 안에 위 클래스들이 있으면, Spring은 발송 구현의 빈을 찾아 서비스 생성자에 전달할 수 있습니다. 서비스는 발송 구현을 직접 생성하지 않고 `MessageSender`가 제공하는 동작을 사용합니다.

일반 Java 코드에서도 같은 생성자에 다른 구현을 전달할 수 있습니다.

```java
MessageSender sender = new EmailMessageSender();
NotificationService service = new NotificationService(sender);
String result = service.sendNotification("예약 안내");
```

## 테스트와 구현체 선택

단위 테스트에서는 고정된 결과를 반환하거나 예외를 던지는 테스트 대역을 생성자에 전달해 서비스의 처리를 확인할 수 있습니다. 실제 외부 시스템과의 통신은 별도의 통합 검증으로 확인합니다.

동일한 인터페이스의 빈이 여러 개라면 어떤 빈을 주입할지 결정할 조건이 필요할 수 있습니다. 특정 주입 지점에서 후보를 좁히려면 `@Qualifier`를 사용하고, 단일 값 주입의 기본 후보를 지정하려면 `@Primary`를 사용할 수 있습니다.

생성자 매개변수에 `@Qualifier`가 필요한 경우, Lombok이 생성한 코드에 어노테이션이 어떻게 반영되는지 확인하거나 생성자를 직접 작성해 주입 지점을 명시합니다.

## 사용 시 확인할 제약

- 서비스 A가 B를, B가 다시 A를 생성자로 요구하면 생성 순서를 정할 수 없는 순환 참조가 생길 수 있습니다. 책임과 의존 방향을 함께 검토합니다.
- 싱글턴 서비스에 의존 객체를 주입하는 것과 서비스의 요청별 상태를 안전하게 관리하는 것은 별개입니다. 사용자 ID처럼 요청마다 다른 값을 공유 필드에 저장하면 동시 요청에서 섞일 수 있습니다.
- 의존성이 많아 생성자가 지나치게 길어지면 서비스가 맡은 책임의 범위를 살펴봅니다.

## 참고 자료와 확인 범위

- [Spring 의존성 주입](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Spring 자동 주입과 단일 생성자](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [Spring 주입 후보 선택](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)

공식 문서를 바탕으로 개념을 보완한 학습 노트입니다. 코드 예시는 Spring 애플리케이션으로 실행 검증하지 않았습니다.
