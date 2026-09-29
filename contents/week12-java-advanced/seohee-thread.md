# Week12 Java 심화 — Thread

> 발표자: 서희  
> 발표일: 2026.09.29  
> 주차 Issue: [#17](https://github.com/jeaminlim0000/eureka-cs-study/issues/17)  
> 발표 질의응답: [Discussion #18](https://github.com/jeaminlim0000/eureka-cs-study/discussions/18)

## 1. Thread란 무엇인가

**Thread**는 프로세스 안에서 실제 코드를 실행하는 흐름입니다. 웹 서버는 요청이 들어오면 스레드를 사용해 컨트롤러·서비스·리포지터리 작업을 처리합니다.

하나의 Java 프로세스 안의 스레드는 Heap 같은 프로세스 자원을 함께 사용하지만, 메서드 호출 순서와 지역 변수는 각 스레드의 Stack에 따로 저장합니다. 그래서 여러 요청을 동시에 처리할 수 있지만, 같은 객체나 같은 데이터에 동시에 접근하면 동시성 문제가 생길 수 있습니다.

```text
Spring 서버 프로세스
 ├─ 요청 스레드 A: 주문 요청 처리
 ├─ 요청 스레드 B: 주문 요청 처리
 └─ Heap: 두 스레드가 함께 접근할 수 있는 객체
```

## 2. Thread Pool과 요청 처리

Tomcat 같은 서버는 요청마다 무제한으로 새 스레드를 만들기보다 **Thread Pool**에서 준비된 요청 스레드를 가져와 사용합니다.

```text
요청 도착 → Thread Pool에서 스레드 획득 → Controller/Service 처리 → 스레드 반환
```

스레드 수가 너무 적으면 요청이 대기하고, 너무 많으면 스레드별 Stack 메모리 사용량과 Context Switching 비용이 증가합니다. 따라서 Thread Pool 크기만 크게 늘리는 것은 근본적인 성능 해결책이 아닙니다.

## 3. 외부 API가 느릴 때 생기는 문제

주문 서비스가 외부 결제 API의 응답을 기다리는 동안, 요청 스레드는 다른 요청을 처리하지 못할 수 있습니다. 결제 API가 5초 동안 느리면 많은 요청 스레드가 5초씩 점유되고, 결국 Thread Pool과 대기 큐가 차 새 요청도 늦어질 수 있습니다.

이때 필요한 대응은 다음과 같습니다.

- 외부 API 연결·응답 타임아웃 설정
- 실패를 빠르게 반환하거나 재시도 정책 적용
- 알림처럼 주문 성공에 필수가 아닌 후처리는 비동기로 분리
- 대기 큐 크기와 동시 요청 수 제한

비동기 또는 Non-Blocking 방식으로 바꾼다고 외부 결제 API 자체의 5초 응답이 빨라지는 것은 아닙니다. 다만 기다리는 동안 요청 스레드를 계속 점유하지 않아 서버가 다른 작업을 처리할 여유를 얻을 수 있습니다.

## 4. 공유 데이터와 Race Condition

여러 스레드가 같은 재고 수량을 읽고 수정하면 **Race Condition**이 발생할 수 있습니다.

```text
재고 = 1
스레드 A: 재고를 읽음 → 1
스레드 B: 재고를 읽음 → 1
스레드 A: 주문 성공, 재고 차감
스레드 B: 주문 성공, 재고 차감
```

주문은 두 건이 성공했지만 실제 재고는 한 개뿐인 초과 판매가 발생합니다. 재고 확인·차감·저장 과정이 하나의 원자적 작업으로 보장되지 않았기 때문입니다.

단일 서버에서는 `synchronized`나 `Lock`으로 같은 JVM 안의 경쟁을 줄일 수 있습니다. 그러나 서버가 여러 대라면 각 인스턴스의 JVM 락은 서로 공유되지 않으므로, 재고처럼 DB가 기준인 데이터는 DB의 조건부 UPDATE, 비관적 락, 낙관적 락 같은 DB 수준의 제어를 선택해야 합니다.

## 5. 발표 질문

### 외부 결제 API가 느려져 주문 요청이 쌓일 때, Thread Pool 크기만 늘려도 문제가 해결되지 않는 원인과 서버에 미치는 영향을 설명해 주세요.

외부 결제 API의 응답 시간이 길다면 스레드 수를 늘려도 외부 API가 빨라지지는 않습니다. 더 많은 스레드가 응답을 기다리게 되어 Stack 메모리 사용량과 Context Switching 비용이 증가하고, Thread Pool과 대기 큐가 차면 새 요청도 늦어질 수 있습니다. 따라서 타임아웃과 동시 요청 제한을 설정하고, 주문 성공에 필수가 아닌 알림 같은 작업은 비동기로 분리하는 방향을 고려해야 합니다.

## 참고

- [Java Thread API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Thread.html)
- [Spring Boot — Tomcat 설정](https://docs.spring.io/spring-boot/appendix/application-properties/index.html#application-properties.server.server.tomcat.threads.max)
