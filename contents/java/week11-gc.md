# Week11 Java Core — GC

> 발표자: 재민　 발표일: 2026.09.22　 주차 Issue: [#1](https://github.com/jeaminlim0000/eureka-cs-study/issues/1)

## 1. GC 대상 판단

**GC(Garbage Collection)**는 더 이상 도달할 수 없는 객체가 차지하는 메모리를 JVM이 자동으로 회수하는 과정입니다.

GC는 단순히 참조 횟수를 세는 대신, **GC Root에서 참조를 따라 객체까지 도달할 수 있는지**를 확인합니다. 실행 중인 스레드가 사용하는 지역 변수의 참조 등이 출발점이 되며, 살아 있는 클래스의 `static` 필드를 통해서도 객체에 도달할 수 있습니다.

```text
GC Root → 객체 A → 객체 B
          두 객체 모두 도달 가능

GC Root     객체 C ↔ 객체 D
            외부에서 접근할 경로가 없으면 회수 대상
```

객체끼리 서로 참조하더라도 GC Root와 연결이 끊겨 있으면 회수 대상이 될 수 있습니다. 따라서 중요한 기준은 **“어딘가에 참조가 있는가?”가 아니라 “살아 있는 실행 경로에서 접근할 수 있는가?”**입니다.

## 2. Young / Old 영역과 GC 종류

세대별 GC는 **대부분의 객체가 짧게 사용되고 사라진다**는 특성을 이용합니다.

| 구분 | 역할 |
|---|---|
| Young Generation | 새 객체가 주로 할당되는 영역. Eden과 Survivor로 구성 |
| Old Generation | Young에서 살아남아 승격된 객체 등을 관리하는 영역 |

객체는 일반적으로 Eden에 생성되고, Young GC에서 살아남으면 Survivor에 남거나 Old로 이동합니다. Young에서 Old로 이동하는 것을 **승격(Promotion)**이라고 합니다.

```text
객체 생성 → Eden
              ↓ Young GC
              ├─ 도달할 수 없는 객체 → 회수
              └─ 살아 있는 객체 → Survivor 또는 Old
```

대표적인 HotSpot 세대별 GC에서는 Young GC 생존과 관련된 **age**를 관리합니다. 승격 기준에는 age와 Survivor 공간 상태 등이 영향을 주며, 공간이 부족하면 일찍 승격될 수도 있습니다. age는 생성 후 지난 시간을 의미하지 않습니다.

모든 객체가 반드시 같은 이동 경로를 거치는 것은 아닙니다. 예를 들어 G1은 Heap을 여러 Region으로 나누어 관리하고, 큰 객체를 Old에 직접 할당하는 경우도 있습니다.

| GC 종류 | 일반적인 의미 |
|---|---|
| Minor GC / Young GC | Young 영역을 대상으로 하는 수집 |
| Major GC | Old 영역 수집을 가리키는 표현. 문서에 따라 Full GC와 같은 의미로 사용되기도 함 |
| Full GC | Heap 전체를 대상으로 하는 수집 |

**Minor GC가 여러 번 발생하면 Major GC로 바뀌는 것은 아닙니다.** 각 수집은 영역의 상태와 Collector 정책에 따라 발생합니다.

또한 Heap이 완전히 찬 뒤에만 GC가 실행되는 것은 아닙니다. 예를 들어 G1은 메모리 상태를 바탕으로 동시 마킹을 시작하고, 이후 Young과 일부 Old Region을 함께 회수하는 **Mixed GC**를 수행할 수 있습니다.

## 3. Stop-The-World

**Stop-The-World(STW)**는 JVM이 특정 작업을 수행하기 위해 애플리케이션 스레드를 일시 정지하는 것을 말합니다. GC 과정에서도 STW가 발생할 수 있습니다.

```text
요청 처리 → GC의 STW 구간 → 요청 처리 재개
```

Spring 서버에서 STW가 길어지면 요청 처리가 멈추는 시간이 늘어나 API 응답 지연에 영향을 줄 수 있습니다.

다만 **GC 실행 시간 전체가 STW 시간인 것은 아닙니다.** 일부 작업을 애플리케이션과 동시에 수행하는 Collector도 있으므로, GC 횟수뿐 아니라 실제 정지 시간도 확인해야 합니다.

## 4. Java 사례와 자주 하는 실수

자주 하는 실수는 **변수에 `null`을 넣으면 객체가 바로 삭제된다고 생각하는 것**입니다.

다음은 참조 관계를 설명하기 위한 예시입니다.

```java
User user = new User();
User backup = user;

user = null;

// backup을 통해 같은 객체를 계속 사용할 수 있습니다.
System.out.println(backup);

backup = null;
```

`user = null`은 `user`의 참조만 끊습니다. `backup`으로 객체에 접근할 수 있는 동안에는 객체를 계속 사용할 수 있습니다.

이후 다른 참조도 사라져 객체에 도달할 수 없게 되면 회수 대상이 될 수 있지만, **해당 줄에서 즉시 GC가 실행되거나 메모리가 회수되는 것은 아닙니다.**

이 원리는 서버의 캐시에도 적용됩니다. 업무상 사용이 끝난 데이터라도 캐시나 컬렉션이 계속 참조하면 회수되지 않을 수 있습니다. 따라서 GC가 있어도 불필요한 참조가 쌓이면 메모리 누수가 발생할 수 있으며, 캐시의 최대 크기와 만료·삭제 조건을 함께 관리해야 합니다.

## 5. 면접형 질문 대비

**Q. 서로 참조하는 두 객체는 GC로 회수할 수 있나요?**

GC Root에서 두 객체로 도달할 경로가 없다면 회수 대상이 될 수 있습니다. Java의 일반적인 GC는 참조 횟수보다 도달 가능성을 기준으로 판단하므로, 객체끼리 순환 참조한다는 이유만으로 계속 유지되지는 않습니다.


## 6. 종합 사례 — 트래픽 증가 후 응답 지연과 OOM

> 트래픽이 증가한 Spring 서버에서 응답 시간이 길어지고 `OutOfMemoryError`가 발생했을 때, GC 관점에서 가능한 원인과 해결 방향은 무엇인가?

트래픽이 증가하면 요청 처리에 필요한 객체 생성량이 늘어 Young GC가 잦아질 수 있습니다. 처리 대기 중인 요청이나 캐시가 객체를 오래 유지하면 Old로 승격되는 객체와 살아 있는 데이터의 양도 증가할 수 있습니다.

이 과정에서 GC에 사용하는 CPU 시간이나 STW 시간이 증가하면 응답 지연에 영향을 줄 수 있습니다. 회수 후에도 새 객체를 할당할 공간이 부족하고 Heap을 더 확장할 수 없다면 `OutOfMemoryError: Java heap space`가 발생할 수 있습니다.

| 확인할 내용 | 대응 방향 |
|---|---|
| 응답 지연 시점의 GC 빈도와 정지 시간 | GC가 실제 지연 원인인지 확인 |
| 객체 할당량과 대량 처리 구간 | 불필요한 임시 객체 생성과 한 번에 처리하는 데이터량 검토 |
| GC 이후에도 계속 증가하는 메모리 사용량 | 캐시·컬렉션·대기 작업이 참조를 유지하는지 확인 |
| Heap dump의 객체와 참조 경로 | 불필요한 참조 제거, 캐시 크기·만료 정책 적용 |
| OOM 상세 메시지와 메모리 설정 | 부족한 영역을 구분하고 실제 사용량에 맞게 조정 |

`OutOfMemoryError`가 모두 Heap 누수를 의미하는 것은 아닙니다. Metaspace 등 다른 메모리 영역에서도 발생할 수 있으므로 **상세 메시지와 메모리 상태를 먼저 확인해야 합니다.**

## 참고 자료

- [Oracle — Generations](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/generations.html)
- [Oracle — Sizing the Generations](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html)
- [Oracle — G1 Garbage Collector](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html)
- [Java Language Specification §12.6.1 — 객체의 도달 가능성](https://docs.oracle.com/javase/specs/jls/se21/html/jls-12.html#jls-12.6.1)
- [Oracle — Troubleshoot Memory Leaks](https://docs.oracle.com/en/java/javase/21/troubleshoot/troubleshooting-memory-leaks.html)
