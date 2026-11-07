# 유레카 CS 스터디

CS 개념을 학습한 기록과, 이를 바탕으로 면접 답변을 연습하는 과정을 함께 관리합니다.

<details>
<summary><strong>학습 1~15주차 · 개념 조사와 발표 기록</strong></summary>

매주 4명이 각자 배정받은 CS 주제를 조사·발표하고, 질문과 답변을 함께 기록합니다.

## 학습 기록

- [주차별 주제와 발표 기록](https://github.com/jeaminlim0000/eureka-cs-study/issues)
- [발표 질의응답](https://github.com/jeaminlim0000/eureka-cs-study/discussions)

## 매주 진행 방식

1. 팀장이 **주차별 이슈 하나**에 4명의 주제와 발표일을 등록합니다.
2. 각자 조사한 자료를 주제별 Markdown 파일로 작성하고 이슈에 링크를 연결합니다.
3. **매주 4명 모두 발표**합니다.
4. 발표마다 Discussion을 하나씩 만들고, 해당 주차 이슈에 연결합니다.
5. 궁금한 점은 **질문 하나당 댓글 하나**로 남깁니다. 한 발표에 여러 질문을 자유롭게 남길 수 있습니다.
6. 답변과 추가 질문은 해당 댓글의 답글로 이어갑니다. 현장에서 답변한 질문도 기록합니다.
7. 발표자는 정정 사항을 자료에 반영합니다. 기록을 정리하면 주차 이슈를 닫고, 남은 질문은 후속 확인 항목으로 연결합니다.

## 주차 이슈 작성

Issues → New issue → **주차별 CS 스터디** 양식을 선택합니다.

| 발표자 | 분야 | 주제 | 발표 자료 | Discussion |
| --- | --- | --- | --- | --- |
| 담당자 1 | | | | |
| 담당자 2 | | | | |
| 담당자 3 | | | | |
| 담당자 4 | | | | |

## Discussion 작성

일반 토론 형식의 **발표 질의응답** 카테고리를 사용합니다.

- 제목: `[Week 01][OS] 프로세스와 스레드 — 발표자`
- 본문: 발표자, 발표일, 주차 이슈 링크, 발표 자료 링크, 핵심 요약
- 댓글: 질문 하나당 새 댓글 / 답변은 해당 댓글의 답글

발표 자료에도 Discussion 링크를 넣어 자료와 대화를 함께 찾아볼 수 있게 합니다.

## 자료 정리

자료는 `contents/os/`, `contents/network/`, `contents/database/`처럼 분야별로 저장합니다. 각 문서에 작성자·발표일·주차 이슈·Discussion·참고 자료를 표시합니다.

이전 8인 노션 스터디 기록은 `archive/`에 질문과 사람별 답변을 보존해 옮깁니다.

</details>

<details>
<summary><strong>면접 대비 1~8주차 · 개념 복습과 면접 답변 연습</strong></summary>

## 진행 목표

1~15주차에서 공부한 개념을 복습하고, 직접 작성한 코드와 실무 상황에 연결해 면접처럼 설명합니다. 매주 **공통 주제 3개를 세 명 모두 공부**합니다.

## 8주 공통 주제

| 주차 | 범위 | 공통 주제 3개 |
| --- | --- | --- |
| [1주차](interview-prep/weeks/week-01-java-spring-di/README.md) | Java 객체지향과 DI | 인터페이스와 다형성 / IoC와 생성자 주입 / Spring 빈 등록과 주입 대상 선택 |
| [2주차](interview-prep/weeks/week-02-data-structures/README.md) | 자료구조와 Java 컬렉션 | 시간 복잡도와 배열·연결 리스트 / 스택·큐·우선순위 큐 / `HashMap`과 `equals()`·`hashCode()` |
| [3주차](interview-prep/weeks/week-03-os-jvm/README.md) | 운영체제와 JVM | 프로세스·스레드와 컨텍스트 스위칭 / 가상 메모리와 페이지 폴트 / JVM 스택·힙과 GC |
| [4주차](interview-prep/weeks/week-04-concurrency/README.md) | 동시성 | 경쟁 상태·원자성·가시성 / 락과 데드락 / 스레드 풀과 대기열 |
| [5주차](interview-prep/weeks/week-05-network-http/README.md) | 네트워크와 HTTP | DNS·TCP·TLS와 HTTPS 요청 흐름 / HTTP 메서드·상태 코드·멱등성 / 인증·인가와 쿠키·세션·JWT |
| [6주차](interview-prep/weeks/week-06-database/README.md) | 데이터베이스 | 조인과 제약 조건 / 인덱스와 실행 계획 / 트랜잭션·격리 수준·락 |
| [7주차](interview-prep/weeks/week-07-spring-jpa/README.md) | Spring과 JPA | Spring MVC 요청 흐름과 예외 처리 / `@Transactional`과 롤백 / 영속성 컨텍스트와 N+1 |
| [8주차](interview-prep/weeks/week-08-troubleshooting/README.md) | 운영 장애와 종합 복습 | 응답 시간·p95/p99·오류율 / DB 커넥션 풀과 외부 API 타임아웃 / 로그와 트레이스로 지연 구간 찾기 |

## 진행 방식

1. 세 주제를 모두 공부하고 각자 핵심 개념과 예시를 Markdown 파일 하나에 정리합니다.
2. 각자 주제별 Discussion 글을 하나씩 올리고, 글마다 기본 질문 1개와 응용 질문 1개를 준비합니다.
3. 다른 두 명이 먼저 답하고, 출제자는 답변에서 더 확인할 부분을 꼬리 질문으로 이어갑니다.
4. 두 명의 답변이 올라오면 AI 참고 답변을 공유하고, 각자 놓친 조건과 수정한 답변을 댓글로 남깁니다.
5. 막혔던 질문은 다음 주에 자료 없이 다시 답하고, 2~3주 뒤에는 조건을 바꿔 한 번 더 답합니다.

## 자료와 질문 찾기

- [면접 대비 8주 전체 안내와 템플릿](interview-prep/README.md)
- [면접 1주차 질문과 예시 답변](interview-prep/weeks/week-01-java-spring-di/interview.md)
- [세 주제를 모두 정리한 개념 노트 예시](interview-prep/weeks/week-01-java-spring-di/notes/example.md)
- 1주차 Discussion — [인터페이스와 다형성](https://github.com/jeaminlim0000/eureka-cs-study/discussions/30) · [생성자 주입](https://github.com/jeaminlim0000/eureka-cs-study/discussions/31) · [Spring 빈](https://github.com/jeaminlim0000/eureka-cs-study/discussions/32)

</details>
