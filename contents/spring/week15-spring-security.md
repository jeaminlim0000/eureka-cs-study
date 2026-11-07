# Week15 · Spring Security 요청 흐름

- 작성자 · 서희 (@superparkgit)
- 학습 일정 · 2026.10.20, 주차 이슈 기준
- 기록 보완일 · 2026.11.07
- 주차 이슈 · [Security 인증 & 보안 #29](https://github.com/jeaminlim0000/eureka-cs-study/issues/29)

## 핵심 개념

인증은 요청자가 누구인지 확인하는 과정이고, 인가는 그 요청자가 특정 작업을 수행할 수 있는지 판단하는 과정입니다.

Servlet 애플리케이션에서 Spring Security는 주로 필터를 통해 요청을 처리합니다. 요청이 Controller까지 전달되기 전에 인증 정보와 접근 정책을 확인하거나 응답을 끝낼 수 있습니다.

## 주요 구성 요소

| 요소 | 역할 |
| --- | --- |
| `DelegatingFilterProxy` | Servlet 컨테이너의 필터 호출을 Spring이 관리하는 필터 빈에 연결 |
| `FilterChainProxy` | 요청에 적용할 보안 필터 체인을 선택하고 실행 |
| `SecurityFilterChain` | 해당 요청에 적용할 보안 필터 목록과 매칭 조건을 정의 |
| `SecurityContextHolder` | 현재 실행 흐름에서 보안 컨텍스트에 접근하는 지점 제공 |
| `Authentication` | 인증 요청 정보 또는 인증된 주체와 권한 등의 정보 표현 |
| `AuthenticationManager` | 인증 처리를 맡는 인터페이스 |
| `AuthenticationProvider` | 지원하는 인증 방식에 따라 인증을 수행 |

## 일반적인 요청 흐름

```text
HTTP 요청
→ Servlet 필터 체인
→ DelegatingFilterProxy
→ FilterChainProxy
→ 요청에 매칭되는 SecurityFilterChain
→ 인증 정보 확인과 접근 권한 판단
→ DispatcherServlet
→ Controller
```

보안 체인이 여러 개라면 순서와 매칭 조건에 따라 처음 매칭된 체인이 선택됩니다. 각 체인의 세부 필터와 인증 처리 방식은 설정에 따라 달라집니다.

인증이 필요한 경우 해당 방식의 인증 필터가 인증 정보를 읽어 인증 처리에 전달할 수 있습니다. 인증 결과를 담은 보안 컨텍스트를 바탕으로 인가 단계가 요청의 접근 조건을 판단합니다.

보안 컨텍스트를 이후 요청에 보존하는 방식은 세션이나 저장소 설정 등에 따라 달라집니다. 같은 사용자의 다음 요청에서 컨텍스트를 어디서 복원하는지도 함께 확인합니다.

## 요청별 접근 정책 예시

다음 코드는 HTTP Basic 인증과 URL별 접근 조건을 설명하는 Servlet 기반 학습 예시입니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/api/public/**").permitAll()
                        .requestMatchers("/api/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
                )
                .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

- `/api/public/**`는 위 인가 규칙에서 인증 없이 접근을 허용합니다.
- `/api/admin/**`는 기본 접두사 설정에서 `ROLE_ADMIN` 권한을 요구합니다.
- 그 밖의 요청은 인증된 사용자를 요구합니다.
- 사용자 정보의 제공 방식과 실제 인증 처리는 별도 빈과 설정을 포함해 구성합니다.

이 예시는 CSRF 설정을 해제하지 않습니다. `permitAll()`은 인가 규칙이므로, 인증·CSRF 등 다른 필터가 처리하는 조건도 함께 적용될 수 있습니다.

## 인증 실패와 접근 거부

일반적인 API 정책에서는 인증이 없거나 유효하지 않은 경우 401, 인증된 사용자에게 권한이 없는 경우 403을 사용할 수 있습니다. 실제 응답은 필터가 실패한 지점과 등록된 핸들러 설정으로 확인합니다.

`ExceptionTranslationFilter`는 아래쪽 필터에서 발생한 인증 관련 예외나 접근 거부를 HTTP 처리로 연결합니다. 필요한 인증을 시작할 때는 `AuthenticationEntryPoint`를, 인증된 사용자의 접근 거부를 처리할 때는 `AccessDeniedHandler`를 사용할 수 있습니다. 개별 인증 필터의 실패 처리도 함께 확인해야 합니다.

## 사용 시 확인할 제약

- 필터에서 끝난 요청은 Controller에 도달하지 않을 수 있습니다. Controller 로그와 예외 처리만으로 실패 경로를 모두 파악하기 어렵습니다.
- URL 수준의 접근 허용과 특정 데이터의 소유자 확인은 각각 확인합니다. 로그인한 사용자도 다른 사용자의 상담 내역을 조회할 권한은 없을 수 있습니다.
- 여러 체인의 순서와 매칭 범위가 겹치면 예상한 체인이 선택되는지 확인합니다.
- 인증을 세션이나 토큰으로 유지하는 방식과 브라우저가 자격 정보를 전달하는 방식에 따라 CSRF 정책을 검토합니다.

## 참고 자료와 확인 범위

- [Spring Security Servlet 아키텍처](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Spring Security 인증 아키텍처](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
- [HTTP 요청 인가](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)

공식 문서를 바탕으로 개념을 보완한 학습 노트입니다. 예시 설정은 실행 검증하지 않았으며, 실제 프로젝트에 적용할 때는 사용 중인 Spring Security 버전과 인증 구성을 함께 확인합니다.
