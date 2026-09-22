# Week11 Java Core — Exception

> 발표자: 서희  
> 발표일: 2026.09.22  
> 주차 Issue: [#1](https://github.com/jeaminlim0000/eureka-cs-study/issues/1)  
> 발표 질의응답: [Discussion #12](https://github.com/jeaminlim0000/eureka-cs-study/discussions/12)

## 1. Error와 Exception

- **Error**는 JVM 또는 시스템 자원에 심각한 문제가 생긴 경우입니다. 예를 들어 `OutOfMemoryError`, `StackOverflowError`가 있습니다. 일반적인 비즈니스 흐름에서 복구 대상으로 처리하지 않습니다.
- **Exception**은 애플리케이션이 예상하거나 처리할 수 있는 문제입니다. 예를 들어 없는 상품을 주문하거나 잘못된 요청값이 들어온 경우입니다.

## 2. Checked Exception과 Unchecked Exception

| 구분 | 특징 | 예시 |
| --- | --- | --- |
| Checked Exception | 컴파일러가 처리 또는 선언을 강제합니다. 외부 자원처럼 호출자가 복구를 선택할 수 있는 경우에 사용합니다. | `IOException` |
| Unchecked Exception | `RuntimeException` 계열입니다. 잘못된 요청, 도메인 규칙 위반, 개발 실수처럼 실행 중 발견되는 문제를 표현합니다. | `IllegalArgumentException`, `ProductNotFoundException` |

Spring 서비스에서는 도메인 규칙 위반을 의미 있는 Unchecked Exception으로 던지고, HTTP 응답 변환은 웹 계층에서 처리하는 방식을 많이 사용합니다.

## 3. Service와 ControllerAdvice의 역할

주문 생성 서비스는 주문 규칙을 처리하고, 문제가 생기면 의미 있는 예외를 던집니다. 이때 서비스가 `ResponseEntity`나 HTTP 상태 코드까지 만들면 서비스가 웹 기술에 묶입니다.

`@RestControllerAdvice`는 여러 컨트롤러에서 발생한 예외를 한곳에서 HTTP 상태와 오류 응답 형식으로 바꿉니다. 그래서 응답 형식이 일관되고, 같은 서비스를 배치 작업이나 메시지 소비자에서도 재사용할 수 있습니다.

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final ProductRepository productRepository;

    @Transactional
    public Long createOrder(Long productId) {
        Product product = productRepository.findById(productId)
                .orElseThrow(() -> new ProductNotFoundException(productId));
        // 주문 생성 규칙 처리
        return 1L;
    }
}

@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handle(ProductNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("PRODUCT_NOT_FOUND", e.getMessage()));
    }
}
```

## 4. 상태 코드 선택 예시

- 요청 형식이나 값이 잘못되면 `400 Bad Request`
- 찾는 상품이 없으면 `404 Not Found`
- 이미 존재하는 값과 충돌하면 `409 Conflict`
- 예상하지 못한 서버 문제면 `500 Internal Server Error`

따라서 주문 생성 중 발생한 모든 예외가 404는 아닙니다. 예외의 의미에 맞는 상태 코드로 변환해야 합니다.

## 5. 발표 질문

### 주문 생성 Service에서 예외가 발생했을 때, Service에서 직접 HTTP 404 응답을 만들지 않고 ControllerAdvice에서 처리하는 이유를 설명해 주세요.

Service는 주문 생성과 재고 확인처럼 비즈니스 규칙을 담당하고, HTTP 상태 코드와 응답 본문은 웹 계층의 책임입니다. 예를 들어 상품이 없으면 Service는 `ProductNotFoundException`을 던지고, `ControllerAdvice`가 이를 받아 404 응답으로 변환합니다. 이렇게 분리하면 Service가 HTTP에 의존하지 않아 배치나 메시지 처리에서도 재사용할 수 있고, 여러 API의 오류 형식도 한곳에서 일관되게 관리할 수 있습니다. 다만 잘못된 요청은 400, 중복 충돌은 409처럼 예외 의미에 맞는 상태 코드를 사용해야 합니다.

## 6. Repository 예외는 어떻게 다루는가

Repository는 DB 접근 문제를 HTTP 응답으로 만들지 않습니다. Spring의 데이터 접근 예외 추상화에 맡기고, Service가 기술 예외를 도메인 의미의 예외로 바꿀 필요가 있을 때만 원인을 보존해 다시 던집니다. 모든 예외를 잡아서 숨기면 장애 원인을 잃을 수 있으므로 피해야 합니다.

## 참고

- [Spring MVC Controller Advice](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-advice.html)
- [Spring Data Access Exception Handling](https://docs.spring.io/spring-framework/reference/data-access/dao.html)

