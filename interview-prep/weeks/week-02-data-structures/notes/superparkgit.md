# 면접 2주차 · 서희 (@superparkgit)

- 정리일: 2026.11.23
- 공통 주제: 시간 복잡도와 배열·연결 리스트 / 스택·큐·우선순위 큐 / HashMap과 객체 동등성

## 1. 시간 복잡도와 배열·연결 리스트

### 핵심 개념

자료구조는 여러 데이터를 저장하고 사용하는 방식입니다. 같은 회원 목록이라도 위치 번호로 읽는지, 이름으로 찾는지, 중간에 삽입하는지에 따라 필요한 작업이 달라집니다.

배열은 생성할 때 길이가 정해집니다. `ArrayList`는 내부 배열을 사용하면서 원소가 늘어나면 필요에 따라 더 큰 배열로 옮깁니다. `LinkedList`는 앞뒤 연결 정보를 가진 노드로 원소를 관리합니다.

```java
List<String> names = new ArrayList<>();
names.add("민지");
names.add("지수");
String second = names.get(1);
```

`List<String>`은 문자열 목록으로 사용할 변수의 타입이고, `names`는 목록 객체를 가리키는 변수입니다. `new ArrayList<>()`로 실제 객체를 만듭니다. `add()`로 원소를 추가하고, 0부터 시작하는 위치 번호인 인덱스로 `get()`을 호출합니다.

### 시간 복잡도

시간 복잡도는 입력 크기가 늘어날 때 필요한 작업량이 어떻게 증가하는지 설명합니다. 아래에서 `n`은 원소 수입니다.

| 표기 | 작업량의 증가 방식 | 예 |
| --- | --- | --- |
| O(1) | 입력 크기에 비례해 늘어나지 않음 | ArrayList의 인덱스 조회 |
| O(n) | 입력 크기에 비례해 늘어남 | 정렬되지 않은 목록에서 끝까지 이름 비교 |
| O(log n) | 탐색 범위 등을 반복해서 줄일 때 나타남 | 정렬된 배열의 이진 탐색 |
| O(n²) | 전체 크기 수준의 반복을 중첩할 때 나타남 | 모든 원소 쌍 비교 |

빅오 표기는 실제 실행 시간을 초 단위로 알려주지는 않습니다. 같은 차수라도 메모리 접근과 구현 방식에 따라 실행 시간이 달라지므로 연산의 종류와 측정 조건을 함께 봅니다.

### 연산별 비교

| 연산 | ArrayList | LinkedList |
| --- | --- | --- |
| 인덱스로 읽기 | O(1) | 중간 위치는 O(n) 순회 필요 |
| 값으로 찾기 | O(n) | O(n) |
| 맨 뒤에 추가 | 분할 상환 O(1), 배열 확장 시 O(n) | O(1) |
| 중간 인덱스에 삽입·삭제 | 원소 이동 때문에 O(n) | 위치 탐색을 포함하면 O(n) |
| 이미 도달한 노드의 연결 변경 | 배열 위치에 따른 이동 필요 | 반복자가 해당 위치를 확보한 경우 연결 변경 자체는 O(1) |

분할 상환 O(1)은 여러 번의 추가에 드는 전체 비용을 나눠 본 표현입니다. `ArrayList`의 매번 추가가 반드시 같은 비용이라는 뜻으로 설명하지 않습니다.

### 코드에서 연결되는 부분

`LinkedList`를 인덱스 기반 반복문에서 `get(i)`로 읽으면, 각 원소를 찾기 위해 순회를 반복해 전체가 O(n²) 수준이 될 수 있습니다. 반복자를 사용하는 순회는 원소를 차례로 따라가므로 전체 O(n)으로 설명할 수 있습니다.

회원 목록을 순서대로 담아 반환하는 경우에는 추가·조회·순회의 빈도를 기준으로 구현을 선택합니다. 중간 삽입이 있다는 사실만으로 `LinkedList`를 선택하지 않고 위치를 찾는 비용까지 확인합니다.

### 참고 자료

- [Java ArrayList](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)
- [Java LinkedList](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)

## 2. 스택·큐·우선순위 큐

### 핵심 개념과 동작 원리

| 구조 | 꺼내는 순서 | 적용할 수 있는 예 |
| --- | --- | --- |
| 스택 | 마지막에 넣은 것을 먼저 꺼냄, LIFO | 실행 취소 이력 |
| FIFO 큐 | 먼저 넣은 것을 먼저 꺼냄 | 접수 순서대로 작업 처리 |
| 우선순위 큐 | 비교 기준상 앞서는 것을 먼저 꺼냄 | 긴급도가 높은 작업 우선 처리 |

Java에서는 `Deque`의 양 끝 연산으로 스택이나 FIFO 큐를 표현할 수 있습니다. `ArrayDeque`는 그 구현 중 하나입니다. `Queue` 타입의 모든 구현이 FIFO로 동작하는 것은 아니므로 실제 구현의 순서 규칙을 확인합니다.

### 코드 예시

```java
Deque<String> history = new ArrayDeque<>();
history.push("이름 수정");
history.push("연락처 수정");
String undoTarget = history.pop(); // 연락처 수정

Queue<String> waiting = new ArrayDeque<>();
waiting.offer("민지");
waiting.offer("지수");
String nextMember = waiting.poll(); // 민지
```

`ArrayDeque`를 사용하는 이 예시에서는 `poll()`과 `peek()`가 비어 있을 때 `null`을 반환합니다. `pop()`은 비어 있으면 예외를 던지므로 빈 상태 처리도 확인합니다.

### 우선순위 큐의 조건

`PriorityQueue`는 기본적으로 비교 기준상 가장 작은 원소를 먼저 꺼냅니다. `Comparator`로 긴급도와 같은 업무 기준을 정할 수 있습니다. `offer()`와 `poll()`은 O(log n), 맨 앞을 확인하는 `peek()`는 O(1)입니다.

동일한 우선순위끼리의 접수 순서는 자동으로 보장되지 않습니다. 긴급도 다음에 접수 순번을 비교하도록 기준을 정하면 동률의 순서를 표현할 수 있습니다. 큐에 넣은 뒤 정렬 기준으로 쓰는 값을 바꾸면 기대한 순서가 유지되는지 별도 처리가 필요합니다.

우선순위 큐를 반복문으로 순회한 결과는 전체 정렬 순서가 보장되지 않습니다. 우선순위 순서로 꺼내려면 `poll()`을 반복하는 동작과 구분합니다.

### 참고 자료

- [Java Deque](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Deque.html)
- [Java ArrayDeque](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayDeque.html)
- [Java PriorityQueue](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PriorityQueue.html)

## 3. HashMap과 equals()·hashCode()

### 핵심 개념

`Map`은 키와 값을 연결합니다. `HashMap`에서 키는 인덱스와 구분합니다. 회원 번호를 키로 사용하면, 그 번호에 연결된 회원 이름을 조회할 수 있습니다.

```java
Map<Long, String> memberNames = new HashMap<>();
memberNames.put(101L, "민지");
memberNames.put(102L, "지수");
String name = memberNames.get(101L);
```

같은 키로 `put()`을 다시 호출하면 해당 키의 값이 교체됩니다. 일반적인 `HashMap`은 순회 순서를 보장하지 않습니다.

### 키를 찾는 과정과 동등성 규약

1. 키의 해시값을 이용해 살펴볼 저장 위치인 버킷을 좁힙니다.
2. 해당 위치의 후보 중 키가 같은지 확인합니다.
3. 일치하는 키의 값을 반환합니다.

참조형 변수의 `==`는 같은 객체를 가리키는지 비교합니다. `equals()`는 클래스에서 정한 동등성을 비교하며, 기본 `Object.equals()`는 객체 동일성을 기준으로 합니다.

`equals()`로 같은 두 객체는 같은 `hashCode()`를 반환해야 합니다. 서로 다른 객체의 해시값이 같을 수도 있으며 이를 해시 충돌이라고 합니다. 직접 동등성을 정의할 때는 두 메서드의 규약을 함께 지킵니다.

### 객체 키 예시

```java
record MemberKey(long memberId) {}

Map<MemberKey, String> namesByKey = new HashMap<>();
namesByKey.put(new MemberKey(101L), "민지");
String name = namesByKey.get(new MemberKey(101L));
```

이 예시의 `record`는 구성 요소인 `memberId`를 기준으로 동등성과 해시값을 제공합니다. 서로 새로 만든 키 객체여도 같은 회원 번호면 같은 키로 취급할 수 있습니다.

키를 넣은 뒤 `equals()`나 `hashCode()` 계산에 사용하는 필드를 바꾸면 저장 위치와 조회 기준이 어긋날 수 있습니다. 키에는 변경되지 않는 회원 ID나 위와 같은 불변 구성의 객체를 사용할 수 있습니다. 참조 필드가 `final`이어도 그 필드가 가리키는 객체의 변경 가능성은 별도로 확인합니다.

### 성능을 설명할 때의 조건

해시가 적절히 분산되는 조건에서 기본 조회·저장은 평균적으로 O(1)을 기대할 수 있습니다. 충돌, 크기 확장, 키의 해시 계산 비용 등을 포함한 모든 호출이 항상 일정한 비용이라고 설명하지 않습니다.

### 참고 자료

- [Java HashMap](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html)
- [Java Object 동등성과 해시 규약](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Object.html)
- [Java Record](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html)

## 확인 범위

코드는 개념을 설명하기 위한 학습용 예제이며, 실제 프로젝트 경험이나 실행 검증 결과를 나타내지 않습니다. 면접 질문과 이후 답변·피드백은 주제별 Discussion에 기록합니다.
