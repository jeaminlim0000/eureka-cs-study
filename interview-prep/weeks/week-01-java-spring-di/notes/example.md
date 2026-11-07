# 면접 1주차 · 서희의 개념 노트 작성 예시

공통 주제 세 개를 한 파일에 정리한 예시입니다. 알림 발송 방식이 달라지는 학습용 코드를 통해 각 개념이 동작하는 지점을 함께 정리했습니다.

작성자는 **서희 (@superparkgit)**입니다. 기존 학습은 [13주차 Spring Core 이슈 #27](https://github.com/jeaminlim0000/eureka-cs-study/issues/27)과 연결합니다.

## 1. 인터페이스와 다형성

### 핵심 개념

인터페이스는 구현 객체가 제공할 동작의 계약을 표현합니다. 호출하는 코드는 인터페이스 타입을 사용하고, 실행 시에는 실제 객체의 구현이 동작할 수 있습니다.

### 예시

```java
interface NotificationSender {
    String send(String message);
}

class EmailNotificationSender implements NotificationSender {
    @Override
    public String send(String message) {
        return "이메일 예시 · " + message;
    }
}

class SmsNotificationSender implements NotificationSender {
    @Override
    public String send(String message) {
        return "문자 예시 · " + message;
    }
}

class NotificationService {
    private final NotificationSender notificationSender;

    NotificationService(NotificationSender notificationSender) {
        this.notificationSender = notificationSender;
    }

    String notifyUser(String message) {
        return notificationSender.send(message);
    }
}
```

`NotificationService`가 `NotificationSender` 타입을 사용해도, 실제로 이메일 구현 객체가 들어오면 그 객체의 `send()`가 실행됩니다. 문자 구현 객체로 바꾸면 서비스 메서드를 수정하지 않고 다른 구현을 실행할 수 있습니다.

## 2. IoC와 생성자 주입

### 핵심 개념

IoC는 객체 생성과 연결 등의 제어를 컨테이너 같은 외부 구성 요소에 맡기는 관점입니다. DI는 필요한 의존 객체를 외부에서 전달하는 방식이며, Spring IoC 컨테이너가 객체의 의존성을 연결하는 데 사용합니다.

### 예시의 생성 흐름

```java
NotificationSender sender = new EmailNotificationSender();
NotificationService service = new NotificationService(sender);
String result = service.notifyUser("예약 안내");
```

위 코드는 호출하는 쪽이 객체를 만들고 생성자로 전달하는 예시입니다. 생성자 주입은 일반 Java 코드에서도 사용할 수 있습니다. Spring을 쓰면 빈 등록 정보에 따라 컨테이너가 객체를 만들고 의존성을 전달합니다.

### 생성자로 받는 이유

- 서비스 생성에 필요한 의존성이 생성자 매개변수로 드러납니다.
- `final` 필드에 한 번 할당해 참조의 재할당을 제한할 수 있습니다.
- 단위 테스트에서는 Spring 컨테이너 없이도 테스트용 구현을 전달할 수 있습니다.

생성자 매개변수에 `null`을 직접 전달하는 것은 가능하므로, 일반 Java 객체에서 생성자 주입만으로 모든 잘못된 입력을 막는다고 설명하지 않습니다.

### 테스트에서 확인할 수 있는 것

고정된 응답을 반환하는 `NotificationSender`를 전달해서, 서비스가 발송 객체에 올바른 메시지를 넘기는지 확인할 수 있습니다. 실제 메일 서버와의 통신은 별도의 통합 검증이 필요합니다.

## 3. Spring 빈 등록과 주입 대상 선택

### 핵심 개념

Spring 빈은 컨테이너가 생성과 의존성 연결 등을 관리하는 객체입니다. 컴포넌트 스캔 대상으로 등록하거나 `@Bean` 메서드 등으로 등록할 수 있습니다.

위의 예시를 Spring 빈으로 사용한다면 이메일과 문자 구현을 각각 등록하고, 서비스의 생성자에서 사용할 구현을 지정할 수 있습니다.

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
class NotificationService {
    private final NotificationSender notificationSender;

    NotificationService(
            @Qualifier("emailNotificationSender") NotificationSender notificationSender
    ) {
        this.notificationSender = notificationSender;
    }

    String notifyUser(String message) {
        return notificationSender.send(message);
    }
}
```

이 예시는 구현 클래스가 컴포넌트 스캔 등으로 각각 빈으로 등록되어 있고, 이메일 구현의 빈 이름이 `emailNotificationSender`인 상황을 전제로 합니다. 생성자가 하나인 경우 Spring은 `@Autowired` 없이도 생성자를 사용할 수 있습니다.

### 구현체가 여러 개라면

- `@Qualifier`는 주입 지점에서 필요한 후보를 좁힐 때 사용합니다.
- `@Primary`는 단일 객체를 주입할 후보가 여러 개일 때 우선할 빈을 지정할 때 사용합니다.
- 오류가 나면 실제로 등록된 빈, 생성자 매개변수 타입, 후보를 고르는 설정을 함께 확인합니다.

Lombok의 `@RequiredArgsConstructor`는 생성자를 생성합니다. Spring의 빈 주입은 그 생성자를 사용하는 별도의 단계입니다. 위 예시에서는 생성자 매개변수의 `@Qualifier`를 보여주기 위해 생성자를 직접 적었습니다.

## 확인 자료와 검증 상태

- [Spring 의존성 주입](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
- [Spring 생성자 자동 주입](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [Spring 주입 후보 선택](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)

코드는 개념 설명을 위한 예제이며, 이 문서를 작성하면서 Spring 애플리케이션 실행 검증을 수행하지 않았습니다.
