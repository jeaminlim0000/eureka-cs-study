# Week14 · Spring Boot

- 작성자 · 서희 (@superparkgit)
- 학습 일정 · 2026.10.13, 주차 이슈 기준
- 기록 보완일 · 2026.11.07
- 주차 이슈 · [Spring MVC & JPA #28](https://github.com/jeaminlim0000/eureka-cs-study/issues/28)

## 핵심 개념

Spring Framework는 IoC·DI, 트랜잭션, MVC 같은 애플리케이션 기반 기능을 제공합니다. Spring Boot는 Spring 애플리케이션을 구성하고 실행하기 위한 자동 설정, 의존성 구성, 실행 지원 등을 제공합니다.

Spring Boot를 사용해도 Controller, Service, Repository의 책임과 비즈니스 규칙은 애플리케이션에서 설계합니다. 자동 설정이 실제로 무엇을 구성했는지도 조건과 설정을 통해 확인할 수 있습니다.

## 주요 구성 요소

| 요소 | 역할 |
| --- | --- |
| Starter | 웹이나 데이터 접근 같은 용도에 필요한 의존성 구성을 묶어 제공 |
| 의존성 관리 | 관리 대상 라이브러리의 버전을 함께 조정해 의존성 조합을 구성하기 쉽게 함 |
| 자동 설정 | 클래스패스, 설정값, 등록된 빈 같은 조건을 보고 애플리케이션 구성을 시도 |
| 내장 서버 | Servlet 웹 애플리케이션을 외부 WAS 배포 없이 실행할 수 있는 구성 제공 |
| 외부 설정 | 코드와 실행 환경의 설정을 분리해 여러 환경에서 같은 애플리케이션을 구성 |

## 애플리케이션 시작점

```java
package com.example.study;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class StudyApplication {
    public static void main(String[] args) {
        SpringApplication.run(StudyApplication.class, args);
    }
}
```

`@SpringBootApplication`은 Spring Boot 설정 클래스, 자동 설정 활성화, 컴포넌트 스캔의 역할을 결합합니다. 메인 클래스를 애플리케이션의 루트 패키지에 두면 하위 패키지의 컴포넌트를 스캔하는 구조를 만들기 쉽습니다.

예를 들어 메인 클래스가 `com.example.study`에 있다면, `com.example.study.notification`의 서비스가 기본 컴포넌트 스캔 범위에 들어갈 수 있습니다. 패키지를 옮긴 뒤 빈이 등록되지 않으면 메인 클래스 위치와 스캔 설정을 함께 확인합니다.

## 자동 설정이 적용되는 과정

1. 추가된 라이브러리와 실행 환경의 조건을 확인합니다.
2. 자동 설정 후보의 조건에 맞는 구성을 적용합니다.
3. 사용자 정의 빈의 존재 등을 조건으로 하는 자동 설정은 해당 빈이 등록되면 적용되지 않을 수 있습니다.
4. 어떤 구성이 선택됐는지는 조건 평가 결과와 실제 등록된 빈으로 확인합니다.

예를 들어 특정 `DataSource` 빈이 없을 때 적용되는 자동 설정은 사용자가 `DataSource` 빈을 제공하면 물러날 수 있습니다. 모든 자동 설정을 사용자 빈 하나가 일괄 대체한다고 설명하지 않고, 해당 자동 설정의 조건을 확인합니다.

`--debug` 옵션으로 실행하면 조건 평가 보고서를 확인할 수 있습니다. 예상과 다르게 설정됐을 때 의존성, 활성 프로필, 사용자 정의 빈, 자동 설정 조건을 함께 살펴봅니다.

## 외부 설정과 프로필

```yaml
spring:
  application:
    name: cs-study

server:
  port: 8080
```

위와 같은 값을 `application.yaml`에 둘 수 있습니다. 프로필별 설정 파일을 사용하면 개발·테스트·운영처럼 실행 환경에 따른 값을 구성할 수 있습니다.

외부 설정에는 우선순위가 있어서 같은 키가 여러 곳에 있으면 최종값이 달라질 수 있습니다. 파일의 값만 보고 판단하기보다 활성 프로필과 환경 변수, 실행 인자까지 함께 확인합니다. 민감한 접속 정보는 공개 학습 자료의 예시값으로 사용하지 않습니다.

## 요청 처리에서의 위치

```text
애플리케이션 실행
→ 컨텍스트 구성과 빈 생성
→ Servlet 웹 서버 시작
→ HTTP 요청 수신
→ 필터 체인과 DispatcherServlet
→ Controller → Service → Repository
```

이 흐름에서 Spring Boot는 애플리케이션 실행과 구성을 지원합니다. 개별 요청의 라우팅과 인자 변환은 Spring MVC가, 데이터 변경 규칙은 서비스가, DB 접근은 Repository와 데이터 접근 기술이 맡습니다. 일부 빈은 지연 생성될 수 있으므로 모든 객체가 반드시 시작 시점에 만들어진다고 가정하지 않습니다.

## 사용 시 확인할 제약

- Starter 추가만으로 업무에 필요한 접속 정보와 보안 정책이 모두 완성되지는 않습니다. 실제 실행 설정과 조건을 확인합니다.
- 로컬과 운영의 결과가 다르면 프로필과 외부 설정값의 차이부터 비교합니다.
- 설정을 변경한 뒤에는 앱 시작 여부와 해당 설정이 관여하는 기능을 확인합니다. 자동 설정은 업무 동작에 대한 테스트를 대신하지 않습니다.

## 참고 자료와 확인 범위

- [Spring Boot 자동 설정](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html)
- [Spring Boot 애플리케이션 패키지 구조](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)
- [Spring Boot 외부 설정](https://docs.spring.io/spring-boot/reference/features/external-config.html)
- [SpringBootApplication API](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html)

공식 문서를 바탕으로 개념을 보완한 학습 노트입니다. 코드는 실행 구조를 설명하는 예시이며 별도의 Spring Boot 애플리케이션 실행 검증은 수행하지 않았습니다.
