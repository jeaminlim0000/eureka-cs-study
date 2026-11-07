# 1주차 · Java 객체지향과 DI

[전체 목차](../../README.md) · [개념 노트 템플릿](../../templates/notes.md) · [Discussion 템플릿](../../templates/discussion.md) · [면접 질문과 예시 답변](interview.md)

## 세 명이 함께 공부할 주제

1. 인터페이스와 다형성
2. IoC와 생성자 주입
3. Spring 빈 등록과 주입 대상 선택

세 주제 모두를 각자의 `notes/<GitHub-ID>.md` 한 파일에 정리합니다. 각자 주제별 Discussion 글을 하나씩 열고 글마다 기본 질문 1개와 응용 질문 1개를 올립니다.

## 출제 예시

1. **기본** · 서비스가 Repository를 직접 생성하는 코드와 생성자로 받는 코드는 어떻게 다른가요?
2. **내 코드** · 서비스의 `private final` 필드 하나를 고르고, 의존 객체가 어디서 빈으로 등록되고 어떻게 주입되는지 설명해 주세요.
3. **상황** · 같은 인터페이스를 구현한 빈을 추가한 뒤 앱이 시작하지 않습니다. 오류 메시지와 코드에서 무엇부터 확인하겠어요?

**공부할 때 확인할 점** · 컨테이너가 빈을 만들고 연결하는 과정과 Lombok이 생성자를 만드는 과정을 나눠 설명해 봅니다.

## 추가 질문은행

1. 인터페이스와 추상 클래스는 어떤 기준으로 고르나요? 구현체가 하나여도 인터페이스를 둘 이유가 있나요?
2. 오버로딩과 오버라이딩은 어떻게 다른가요? 인터페이스 타입 변수로 호출할 때 어떤 구현이 실행되나요?
3. DI와 의존성 역전 원칙은 어떻게 연결되나요? 구체 클래스를 생성자로 받아도 DI인가요?
4. 생성자 주입, 필드 주입, 세터 주입의 차이는 무엇인가요?
5. 일반 Java 객체와 Spring이 관리하는 빈은 어떻게 다른가요?
6. @Component, @Service, @Repository, @Bean은 각각 언제 쓰나요?
7. @RequiredArgsConstructor는 어떤 생성자를 만드나요? 생성자가 하나일 때 @Autowired가 없어도 어떻게 주입되나요?
8. @Primary와 @Qualifier는 각각 어떤 경우에 쓰나요?
9. 빈 누락, 주입 후보 중복, 순환 참조를 어떻게 구분하나요?
10. Spring 없이 서비스 단위 테스트를 할 때 외부 호출 실패를 어떻게 재현하나요?

## 바로 사용할 수 있는 업로드 예시

- [공통 주제 세 개를 정리한 개념 노트](notes/example.md)
- [주제 1 질문 글](examples/01-interface-question.md)
- [주제 2 질문 글](examples/02-di-question.md)
- [주제 3 질문 글](examples/03-bean-question.md)
- [댓글과 복습 답변 예시](examples/comments.md)

## 기존 학습과 Discussion 연결

- [학습 13주차 Spring Core 이슈 #27](https://github.com/jeaminlim0000/eureka-cs-study/issues/27)
- [주제 1 인터페이스와 다형성](https://github.com/jeaminlim0000/eureka-cs-study/discussions/30)
- [주제 2 IoC와 생성자 주입](https://github.com/jeaminlim0000/eureka-cs-study/discussions/31)
- [주제 3 Spring 빈 등록과 주입 대상 선택](https://github.com/jeaminlim0000/eureka-cs-study/discussions/32)
