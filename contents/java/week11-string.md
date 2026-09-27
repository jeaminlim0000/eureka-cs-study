# Week11 Java Core — String

> 발표자: 나  
> 발표일: 2026.09.27  
> 주차 Issue: [#1](https://github.com/jeaminlim0000/eureka-cs-study/issues/1)  
> 발표 질의응답: [Discussion #12](https://github.com/jeaminlim0000/eureka-cs-study/discussions/12)

## 1. 개념정리

- **String 불변성**: String 객체 수정 시 새로운 객체를 생성 후 반환합니다(메모리 수정이 아님). 이로 인해 멀티스레드-safe(Race Condition 문제가 없음)하며, String pool(캐시) 사용이 가능합니다.
- **String pool**: JVM의 Heap 영역에 위치하며, 리터럴 방식으로 생성 시 같은 문자열의 참조 주소를 반환하고 등록합니다. 
  - 리터럴(Literal) 방식 생성 예: `String s = "hello";`
- **== 와 equals()**: `==`는 참조 주소 비교, `String.equals(값)`는 실제 값 비교입니다.
- **StringBuilder**: 불변성으로 인해 메모리를 많이 차지하는 문제를 해결하는 **가변배열버퍼**입니다. 단, Thread-safe 속성을 잃기 때문에 멀티스레드 환경에서는 **`synchronized`**가 붙은 **`StringBuffer`**로 대체 가능합니다.

## 2. Java/Spring 예시 1개

리터럴 / new(캐시 사용 X) 객체 생성 비교:

```java
public class StringExample {
    public static void main(String[] args) {
        // 1. 리터럴 방식 (String Pool 활용)
        String str1 = "Java";
        String str2 = "Java";
        
        // 2. new 연산자 방식 (항상 Heap에 새로운 객체 생성)
        String str3 = new String("Java");

        System.out.println(str1 == str2);      // true (같은 주소 참조)
        System.out.println(str1 == str3);      // false (다른 주소)
        System.out.println(str1.equals(str3)); // true (값은 동일)
    }
}
```

## 3. 자주 생기는 문제나 실수 1개

이번에 SSE를 진행하면서 생길 수 있는 문제:

```java
while(조건){
    String result = "";
    result += data;
} // 버퍼가 아니기에 더할 때마다 new 된다고 보면 됩니다.
```

- 이는 곧 힙 메모리가 금방 차고 객체의 참조를 잃게 만들어 GC의 오버헤드를 유발하여 서버 지연을 초래합니다.
- 따라서 **`StringBuilder`** / **`StringBuffer`**를 이용해야 합니다.

```java
StringBuilder sb = new StringBuilder();
while(조건){
    sb.append(data); 
}
```

## 4. 면접형 질문 1~2개

### String은 불변인데, 코드상에서 `str = str + " world"` 처럼 값을 바꾸는 것이 어떻게 가능한가요?

값을 바꾸는 것처럼 보이지만, 실제로는 기존 메모리를 수정하는 것이 아닙니다. 내부적으로 기존 문자열과 " world"를 합친 새로운 `String` 객체를 새롭게 생성한 뒤, `str` 변수가 그 새로운 참조 주소를 가리키도록 변경하는 것입니다. 기존 메모리는 GC의 대상이 됩니다.

## 5. 다른 발표자에게 물어볼 질문 1개

- **JVM 담당 (현빈님)**
    - JIT는 무엇을 기준으로 캐시할 것을 정하나요? (그리고 호출되지 않을 코드도 일단 java 파일을 전부 읽고 계산하는 건가요?)
    - JVM은 계층적 또는 순차적으로 컴파일 순서를 갖는지 궁금합니다.
- **GC 담당 (재민님)**
    - 혹시 GC 대상을 어떻게 찾는지 간단히 알려주실 수 있나요? (참조 추적이 안 되는데 어떻게 추적 불가 상태라는 것을 알고 지우는지!)
    - (AGE를 설명 안 하신다면) Young과 Old는 어떻게 구분하나요?
    - Young 영역이 꽉 차서 제일 어린 객체가 Old로 넘어가버리면 생길 문제가 무엇일까요? / 그걸 또 나이순으로 정렬해서 해결하려고 하면 그 추가 작업이 오래 걸리지는 않나요?
- **Exception 담당 (서희님)**
    - 어느 단에서 throw를 하고 catch를 하는 게 바람직한지(throw / catch) 궁금합니다.
        1. 재고 부족 같은 **내부 비즈니스 로직(Local Code) 예외 (Service? / )**
        2. 데이터베이스 제약조건 위반 같은 **Local DB 예외 (repo / )**
        3. 결제 연동 같은 **외부 API 통신 실패 예외 (API 호출 Class / )**
    - 즉 1, 2, 3이 같을 수 없는 상황에서, 이 세 가지 성격이 다른 예외들을 각각 어떤 방식으로(예: 커스텀 예외 생성, 에러 코드 통일 등) 구분해서 관리하고 처리하는 것이 실무적으로 좋은 설계일까요?
    - (제 생각) 그냥 catch는 `@RestControllerAdvice`로 전용 Handler 하나 둬서 여기서 받는 방식도 있지만, 400이나 500 에러 정도를 명확히 구분하기 위해서는 throw에서 service 전까지 한 번 감싸주는(wrap) 작업이 필요해 보입니다. 어떻게 생각하시나요?

## 6. 종합사례 대비

### OOM(OutOfMemory) 발생 시 대응 방안 시나리오

- 결국 3번 문제와 직결됩니다. 무분별한 `String` 더하기 연산 등으로 인해 가비지 객체들이 무한정 쏟아져 힙(Heap) 영역이 금방 차고, GC가 제대로 처리하지 못해 Old 영역까지 밀려가면서 서버 지연 후 OOM이 발생합니다. 결국 `StringBuilder`나 `StringBuffer`를 이용하는 방식으로 원인 코드를 수정해야 합니다.
