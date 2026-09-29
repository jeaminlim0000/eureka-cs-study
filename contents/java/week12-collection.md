# Week12 Java 심화 — Collection

> 발표자: [발표자 이름]　 발표일: 2026.09.29　 주차 Issue: [#[이슈번호]]([이슈 링크])

## 1. 핵심 개념

**Java Collection**은 객체를 관리하는 자료구조(Map 포함)입니다.

| **인터페이스** | **순서/중복 특징** | **주요 구현 클래스** | **특징 및 사용 시점** |
| --- | --- | --- | --- |
| **`List`** | • 순서 유지 (인덱스 존재)<br>• 중복 허용 | • `ArrayList`<br>• `LinkedList`<br>• `Vector` / `Stack` | • `ArrayList`: 내부적으로 연속된 배열 사용. 인덱스 기반 조회($O(1)$)가 매우 빠름.<br>• `LinkedList`: 양방향 노드 연결 구조. 중간 삽입/삭제($O(1)$)가 빈번할 때 유리. |
| **`Set`** | • 순서 없음 (기본)<br>• 중복 불가 (고유값) | • `HashSet`<br>• `LinkedHashSet`<br>• `TreeSet` | • `HashSet`: 해시 테이블 기반, 탐색 속도 최상($O(1)$).<br>• `LinkedHashSet`: 추가된 순서를 기억하는 Set.<br>• `TreeSet`: 이진 탐색 트리(레드-블랙 트리) 기반으로 데이터가 자동 오름차순 정렬됨($O(\log N)$). |
| **`Queue` / `Deque`** | • 대기열 처리<br>• FIFO / LIFO | • `ArrayDeque`<br>• `PriorityQueue`<br>• `LinkedList` | • `ArrayDeque`: 스택과 큐 양쪽으로 쓸 수 있는 고성능 덱(Stack 클래스 대체 권장).<br>• `PriorityQueue`: 우선순위 힙(Heap) 기반으로 우선순위가 높은 데이터가 먼저 나옴. |
| **`Map`** | • 키-값 쌍 관리<br>• Key: 중복 불가<br>• Value: 중복 허용 | • `HashMap`<br>• `LinkedHashMap`<br>• `TreeMap`<br>• `Hashtable` | • `HashMap`: 해시 기반 키-값 저장소 ($O(1)$).<br>• `LinkedHashMap`: 삽입 순서를 유지하는 Map.<br>• `TreeMap`: 키(Key) 기준으로 자동 정렬되는 Map.<br>• `ConcurrentHashMap`: 멀티스레드 동시성 안전 보장. |

**Collections** 클래스는 Java Collection의 편의 기능을 제공합니다 (Map.sort() 제외).

### Interface Collection
Map을 제외한 나머지 Java Collection을 받을 수 있는 인터페이스입니다. **Map(Key-value 구조)** 과 **Collection(단일 원소 구조)** 은 구조가 달라 Interface를 공유할 수 없으며, 메서드 동작 방식(조회, 순환 등)에 차이가 있습니다.

- `add(E e)` / `addAll(Collection<? extends E> c)`: 원소 추가
- `remove(Object o)` / `removeAll(Collection<?> c)`: 원소 삭제
- `contains(Object o)`: 특정 원소 포함 여부 확인 (`boolean`)
- `size()`: 저장된 원소의 개수 반환
- `isEmpty()`: 비어있는지 확인 (`boolean`)
- `clear()`: 모든 원소 제거
- `iterator()`: 컬렉션 순회를 위한 반복자(`Iterator`) 반환
- `stream()`: Java 8 이상에서 함수형 처리를 위한 스트림 생성

### Array[] 와 List(Collections) 차이
- **배열 (Array)**: 원시/특수객체, JVM 직접 관리, 고정 크기, 원시+참조객체 담기 가능
  - 원시타입: 주소가 아닌 값 자체를 담는 타입(Stack), 메서드 없음, null 불가, ex) int
  - 참조타입: Heap의 주소 사용, ex) Integer, 접근 속도 상대적으로 느림
- **List**: Java 일반 클래스, 참조객체만 담기 가능 (원시객체를 담기 위해 **boxing** 필요, ex: `stream().boxed()`), `<T>` 제네릭(컴파일 시점 타입 체크) 지원

## 2. Java / Spring 사용 예시

멀티스레드 환경(Spring Bean)에서의 안전한 Map 사용 예시입니다.

```java
@Service
public class SessionStoreService {
    // 🚨 일반 HashMap 사용 시 동시에 여러 요청이 접근하면 데이터가 유실되거나 에러 발생
    // private final Map<String, String> sessionStore = new HashMap<>(); 
    
    // ✅ 분할 잠금(Lock Stripping) 기법이 적용되어 멀티스레드에서도 안전하고 빠른 ConcurrentHashMap 사용
    private final Map<String, String> sessionStore = new ConcurrentHashMap<>();

    public void saveSession(String sessionId, String userId) {
        sessionStore.put(sessionId, userId);
    }
}
```

- 일반적인 상태를 가지는 `@Service` 빈 등 여러 스레드가 접근할 수 있는 환경에서는 컬렉션의 스레드 안전성이 보장되어야 합니다.
- 위 코드처럼 분할 잠금 기법을 사용하는 `ConcurrentHashMap`을 통해 안전하고 효율적인 상태 관리가 가능합니다.

## 3. 잘못 사용했을 때 생기는 문제

**순회 중 구조 변경 문제 (`ConcurrentModificationException`)**

```text
ArrayList나 HashMap을 for-each 문으로 순회(Iteration)하는 도중에 컬렉션의 요소를 add() 하거나 remove() 할 때 발생하는 실패(Fail-fast) 현상
```

컬렉션의 크기나 구조가 변경되었음을 Iterator가 감지하여 안전을 위해 실패 처리(Fail-fast)를 하기 때문에 예외가 발생합니다.

```java
List<String> list = new ArrayList<>();
list.add("사과");
list.add("바나나");
list.add("포도");

// 바구니에서 과일을 하나씩 꺼내보며 검사하는 중
for (String fruit : list) {
    if (fruit.equals("바나나")) {
        // [사고 발생!] 바구니에서 갑자기 바나나를 쏙 빼버림!
        list.remove(fruit);
    }
}
// 결과: ConcurrentModificationException 에러 발생하며 프로그램 사망!
```

### 어떻게 해결해야 할까? (3가지 방법)

1. **반복자(`Iterator`)에게 직접 부탁하기 (가장 전통적인 방법)**
   단일 스레드에서는 `list.remove()` 대신 `it.remove()`를 사용하여 안전하게 삭제합니다.
2. **`removeIf` 사용하기 (현대 자바 추천 방법, Java 8+)**
   `list.removeIf(fruit -> fruit.equals("바나나"));` 형태로 깔끔하게 처리합니다.
3. **멀티스레드 환경일 때**
   `ConcurrentHashMap`, `CopyOnWriteArrayList` 등 순회하며 수정해도 꼬이지 않도록 내부 설계된 동시성 컬렉션을 사용합니다.

## 4. ThreadSafe 컬렉션 사용 시 주의점

멀티스레드 환경에서 컬렉션 선택 시 락을 거는 단위(Granularity)와 내부 동기화 방식의 차이를 인지해야 합니다.

### `Collections.synchronizedMap` VS `ConcurrentHashMap`

| **비교 항목** | **Collections.synchronizedMap** | **ConcurrentHashMap** |
| --- | --- | --- |
| **락 단위 (Granularity)** | **맵 전체** (단일 객체 락) | **해시 버킷 단위** (노드별 세분화 락) |
| **읽기 (`get`) 성능** | 락 경합 발생 (블로킹) | **락 없음** (`volatile` 기반 완전 논블로킹) |
| **쓰기 (`put`) 성능** | 한 번에 하나의 스레드만 가능 | 서로 다른 버킷이면 **동시 다중 쓰기 가능** (CAS + 버킷 락) |
| **`null` 허용 여부** | Key/Value 모두 **`null` 허용** (원본 Map에 따라 다름) | Key/Value 모두 **`null` 절대 불가** (`NullPointerException`) |
| **순회(Iteration) 시 안전성** | 순회 중 수정 시 `ConcurrentModificationException` 발생 (동기화 블록 수동 처리 필요) | **약한 일치성(Weakly Consistent)** 보장, 순회 중 수정되어도 예외 발생 안 함 |

*순회 시 주의점:* `Collections.synchronizedMap`은 맵 전체에 자동 락이 걸리지 않으므로, 순회 시 **수동으로 `synchronized` 블록**을 감싸야만 합니다.

## 5. 종합 사례 — 주문 서비스 내 동시 재고 차감 문제

> 주문 서비스에서 동시에 다수의 재고 차감 요청이 발생할 경우 어떤 컬렉션 동기화 전략을 취해야 할까?

멀티스레드에서 Thread-Safe한 컬렉션이 필요하다고 하여 모든 메서드에 `synchronized`를 거는 `Vector` 같은 레거시를 사용하는 것은 심각한 병목을 초래합니다. 대신, `volatile`(Lock-Free), 원자적 연산(CAS) 등으로 락 오버헤드를 줄인 멀티스레드 동시성 대체제를 적극 활용해야 합니다.

| **기존 레거시 클래스** | **단일 스레드 대체제 (기본)** | **멀티스레드 동시성 대체제** |
| --- | --- | --- |
| **`Vector`** | **`ArrayList`** | `CopyOnWriteArrayList` 또는 `Collections.synchronizedList()` |
| **`Hashtable`** | **`HashMap`** | **`ConcurrentHashMap`** |
| **`Stack`** (Vector의 자식) | **`ArrayDeque`** | `ConcurrentLinkedDeque` |

## 참고 자료

- [[참고자료명 1]]([URL 링크])
- [[참고자료명 2]]([URL 링크])
