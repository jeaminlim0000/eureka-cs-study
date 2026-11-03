# Week15 Security 인증 & 보안 — Web 보안과 인프라

> 발표자: [최정현]　 발표일: [2026.10.20]　 주차 Issue: [#[이슈번호]](링크)

## 1. 핵심 개념

### 1) 웹 보안 3대 요소 (XSS, CSRF, CORS)

- **XSS (Cross-Site Scripting)**: 웹 페이지에 악성 자바스크립트를 삽입하여 사용자의 브라우저에서 실행되도록 유도하는 공격입니다. (목적: 세션/토큰 탈취, 비정상적 스크립트 실행)
- **CSRF (Cross-Site Request Forgery)**: 사용자가 로그인된 브라우저 상태를 악용하여, 해커가 심어둔 악성 링크나 폼을 통해 **사용자의 의도와 무관하게 서버로 인증된 위조 요청**을 보내게 만드는 공격입니다. (브라우저의 쿠키 자동 전송 특성을 악용)
- **CORS (Cross-Origin Resource Sharing)**:
  1. 브라우저의 기본 보안 정책인 **SOP(Same-Origin Policy, 동일 출처 정책)를 안전하게 예외 허용하기 위한 표준 메커니즘**입니다. (원치 않는 출처에서 온 악의적 스크립트가 API 응답 데이터를 함부로 열람하지 못하도록 브라우저가 차단)
  2. 브라우저는 타 출처로 본 요청을 보내기 전, 사전 검사(**Preflight, `OPTIONS` 메서드**)를 서버에 먼저 찔러보아 안전한 출처인지 확인합니다.

---

### 2) 쿠키 보안 속성 (HttpOnly, Secure, SameSite)

| 속성 | 방어 대상 | 동작 및 특징 |
|---|---|---|
| **`HttpOnly`** | **XSS 방어** | 자바스크립트(`document.cookie`)의 접근을 원천 차단하여 토큰/세션 탈취를 방지합니다. |
| **`Secure`** | **네트워크 스니핑 (중간자 공격, MITM) 방어** | HTTPS(TLS 암호화) 통신 환경에서만 쿠키를 전송합니다. <br>*(TLS는 비대칭키로 Handshake 후 실제 데이터는 대칭키로 빠르게 암·복호화 교환)* |
| **`SameSite`** | **CSRF 방어** | 서드파티(Cross-Site) 사이트에서 유입되는 요청 시 쿠키 전송 범위를 제한합니다. |

- `SameSite=Strict`: 어떤 크로스 사이트 요청에서도 쿠키를 절대 전송하지 않음 (가장 안전).
- `SameSite=Lax`: 기본 권장값. 안전한 탑레벨 탐색(GET 링크 클릭 이동 등)만 전송하고, 폼 전송(POST) 등은 미전송.
- `SameSite=None`: 모든 요청에 전송 (반드시 `Secure` 속성이 필수).

---

### 3) Web Server (Nginx) vs WAS (Tomcat / Spring Boot)

- **Web Server (Nginx)**: **클라이언트의 최전방 대문(Reverse Proxy)**. 정적 리소스 서빙, **SSL/TLS Termination(암호화/복호화 오프로딩)**, 로드 밸런싱, Rate Limiting(DDoS 완화), 공통 보안 헤더 일괄 부여.
- **WAS (Tomcat / Spring Boot)**: 동적 비즈니스 로직 수행, Spring Security 필터 체인 실행, DB 트랜잭션 관리.
- **분리하는 이유**: CPU 부하가 큰 SSL/TLS Handshake 암호화 연산과 정적 리소스 처리를 Nginx가 앞단에서 대신 처리(Offloading)해 줌으로써, 뒷단의 WAS는 비즈니스 로직과 트랜잭션 처리에만 온전히 CPU 자원을 집중할 수 있습니다.

---

## 2. Java / Spring 예시

Refresh Token을 쿠키로 발급할 때 **갱신 엔드포인트(`/api/auth/refresh`)로 스코프(`path`)를 제한**하면, 매 일반 API 호출마다 쓸데없이 쿠키가 실려 가는 네트워크 낭비를 줄이고 공격 표면(Attack Surface)을 크게 줄일 수 있습니다.

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<LoginResponseDto> login(@RequestBody LoginRequestDto request, HttpServletResponse response) {
        // ... 로그인 인증 로직 수행 후 토큰 발급 ...
        String accessToken = "jwt.access.token";
        String refreshToken = "jwt.refresh.token";

        // Refresh Token을 위한 보안 쿠키 생성 (Spring Framework ResponseCookie)
        ResponseCookie cookie = ResponseCookie.from("refreshToken", refreshToken)
                .httpOnly(true)            // XSS 방어 (자바스크립트 접근 불가)
                .secure(true)              // HTTPS 통신에서만 전송
                .sameSite("Strict")        // CSRF 방어 (타 사이트 유입 요청 쿠키 차단)
                .path("/api/auth/refresh") // 갱신 API 호출 시에만 쿠키가 실려가도록 스코프 제한
                .maxAge(Duration.ofDays(7))
                .build();

        response.addHeader(HttpHeaders.SET_COOKIE, cookie.toString());

        // Access Token은 응답 바디(JSON)로 반환 -> 클라이언트 메모리(변수)에서 관리
        return ResponseEntity.ok(new LoginResponseDto(accessToken));
    }
}
```

---

## 3. 자주 생기는 문제나 실수

### 1) Nginx 프록시 환경에서 WAS의 프로토콜 인식 실패 (TLS Termination 이슈)
성능과 통신 간소화를 위해 클라이언트 ↔ Nginx는 HTTPS로 통신하고, Nginx ↔ Spring Boot는 내부망 HTTP(`localhost`)로 통신할 때 주로 발생합니다. Spring Boot는 요청이 HTTP로 들어온 줄 알고 `Secure` 쿠키를 거절하거나 리다이렉트 URL을 `http://`로 만들어 내려보냅니다.

#### 해결 1: Nginx 설정 (원본 요청 정보 헤더 전달)
```nginx
proxy_set_header X-Forwarded-Proto $scheme;                   # "원래 손님이 HTTPS로 들어왔어!"
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for; # "손님의 실제 원본 IP는 이거야!"
proxy_set_header Host              $http_host;                 # "손님이 치고 들어온 원래 도메인 주소야!"
```

#### 해결 2: Spring Boot `application.yml` 설정 (헤더 신뢰)
```yaml
server:
  forward-headers-strategy: native
```
- 내장 톰캣(`RemoteIpValve`)이 Nginx의 `X-Forwarded-Proto` 헤더를 읽어 `request.isSecure() == true`, `request.getScheme() == "https"`로 정상 인식하여 리다이렉트 및 `Secure` 쿠키가 정상 동작합니다.

---

### 2) Preflight(`OPTIONS`) 401 에러
브라우저가 보안 검사를 위해 보내는 **`OPTIONS` 요청에는 Authorization 헤더나 쿠키가 포함되지 않습니다.**
Spring Security에서 `anyRequest().authenticated()`만 걸어두면 시큐리티가 "토큰이 없다"며 401 Unauthorized를 뱉고, 브라우저는 본 요청을 취소해 버립니다.

- **해결책:** `SecurityFilterChain`에 **`.cors(Customizer.withDefaults())`를 선언**하여 시큐리티 필터 최상단(`CorsFilter`)에서 `OPTIONS` 요청을 먼저 가로채 통과시켜 주어야 합니다.

---

## 4. 실무 면접형 질문

> **Q1. JWT 토큰을 브라우저의 `LocalStorage`에 저장하는 것과 `HttpOnly` 쿠키에 저장하는 것의 보안적 트레이드오프는 무엇인가요?**  
> **A.** `LocalStorage`에 저장하면 브라우저의 자바스크립트로 직접 접근할 수 있어 **XSS 공격 시 토큰이 그대로 탈취**당하는 치명적인 취약점이 있습니다. 반면 `HttpOnly` 쿠키는 자바스크립트 접근이 차단되므로 XSS 탈취는 방어할 수 있지만, 브라우저가 요청 시 자동으로 쿠키를 실어 보내는 특성 때문에 **CSRF 공격에 노출**됩니다. 따라서 쿠키 저장 시에는 반드시 `SameSite` 속성과 CSRF 방어 정책을 병행해야 합니다.

> **Q2. Nginx를 리버스 프록시로 두고 SSL Termination을 적용했을 때, Spring Boot에서 `Secure` 쿠키가 클라이언트에게 전달되지 않는 이유와 해결 방법은 무엇인가요?**  
> **A.** Nginx와 Spring Boot 사이의 통신이 내부망 HTTP로 이루어지기 때문입니다. Spring Boot 입장에서는 비보안 연결로 판단하여 `Secure` 속성이 지정된 쿠키 전송을 무시합니다. 이를 해결하려면 Nginx에서 `X-Forwarded-Proto` 헤더에 원래 프로토콜(`$scheme`)을 담아 전달하고, Spring Boot에서 `server.forward-headers-strategy: native`를 활성화하여 톰캣의 `RemoteIpValve`가 해당 헤더를 공식 프로토콜로 인정하도록 설정해야 합니다.

---

## 5. 다른 발표자에게 물어볼 질문 (Spring Core: IoC / DI / `@Component`, `@Bean`)

> **"Spring Security 설정 클래스에서 `SecurityFilterChain`이나 CORS 설정을 빈으로 등록할 때 `@Component` 대신 `@Configuration` 클래스 내부의 `@Bean` 메서드를 주로 사용합니다. 스프링 컨테이너의 싱글톤 보장 메커니즘(CGLIB 프록시의 `proxyBeanMethods`) 관점에서 두 방식의 근본적인 차이는 무엇인가요?"**

---

## 6. 종합 사례 — 소셜 로그인 사용자의 주문 API 호출과 토큰 갱신 시나리오

> "소셜 로그인 사용자가 JWT Access Token으로 주문 API를 호출하는 서비스에서, 사용자 권한이 변경(예: 정지)되었거나 Refresh Token이 쿠키에 저장된 환경일 때 발생할 수 있는 보안 문제와 해결 방향은?"

```text
[클라이언트] -> (HTTPS / Cookie or Bearer) -> [Nginx] -> (HTTP / X-Forwarded-*) -> [Spring Boot WAS]
```

1. **클라이언트 → Nginx → WAS 구간 보안**:
   - 쿠키에 담긴 Refresh Token은 Nginx가 앞단에서 TLS Termination을 처리한 후, `X-Forwarded-Proto` 헤더를 전달해야 WAS에서 `Secure` 쿠키 유효성이 깨지지 않고 안전하게 유지됩니다.
2. **주문 API 호출 시 Access Token 처리**:
   - 주문 API 호출 시에는 요청 헤더의 `Authorization: Bearer <Access_Token>`을 통해 Spring Security가 Stateless하게 인증/인가를 처리합니다. 브라우저가 쿠키처럼 자동으로 실어 보내지 않으므로 이 구간은 CSRF 공격으로부터 안전합니다.
3. **권한 변경 및 토큰 갱신 시의 보안 위험과 대응**:
   - **문제점:** 사용자가 정지 처리되었더라도, 이미 발급된 Stateless한 JWT Access Token은 만료 전까지 서버 DB를 조회하지 않으므로 주문 API를 계속 호출할 수 있는 공백이 발생합니다.
   - **대응 1 (토큰 갱신 시 CSRF 방어):** 새 Access Token을 발급받기 위해 쿠키 기반의 `/refresh` 엔드포인트를 호출할 때, `SameSite=Strict/Lax` 적용 및 Custom Header(`X-Requested-With`) 검증을 거쳐 타 사이트에서의 위조 요청을 원천 차단해야 합니다.
   - **대응 2 (즉각적 권한 박탈 인프라):** 완전한 Stateless의 한계를 보완하기 위해, Redis 기반의 **Token Blacklist** 또는 **User Versioning**(토큰 페이로드의 버전과 Redis의 유저 상태 버전 비교)을 WAS Security Filter에 추가하여 권한 변경 시 즉시 차단할 수 있도록 인프라 레벨의 방어선을 구축해야 합니다.

---

## 참고 자료

- [Spring Security Documentation - CORS & CSRF Protection](https://docs.spring.io/spring-security/reference/servlet/exploits/index.html)
- [MDN Web Docs - Using HTTP cookies (HttpOnly, Secure, SameSite)](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)
- [Nginx Reverse Proxy Documentation](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
