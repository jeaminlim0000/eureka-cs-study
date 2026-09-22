# Week09 DB RDB 기초 — 개인 학습 정리

> 작성자: 서희  
> 발표일: 2026.09.08  
> 관련 Issue: [#10](https://github.com/jeaminlim0000/eureka-cs-study/issues/10)  
> 당시 개별 발표 담당은 미확인으로, 개인 학습 정리로 보관합니다.

## Key

Primary Key는 행을 유일하게 식별하는 키이며 NULL을 허용하지 않습니다. Foreign Key는 다른 테이블의 키를 참조해 데이터 관계와 참조 무결성을 표현합니다.

## Join

Join은 여러 테이블의 관련 행을 연결해 조회하는 방식입니다. `INNER JOIN`은 양쪽 모두 일치하는 데이터만, `LEFT JOIN`은 왼쪽 테이블의 행을 모두 유지하면서 오른쪽 데이터를 연결합니다.

주문과 회원을 조회할 때 `orders.member_id`와 `member.id`처럼 관계가 명확한 키를 기준으로 Join해야 합니다. 잘못된 Join 조건은 행 수가 폭증하는 Cartesian Product를 만들 수 있습니다.

## Normalization

정규화는 중복을 줄이고 수정 이상을 막기 위해 데이터를 역할에 맞는 테이블로 나누는 과정입니다. 예를 들어 주문 행마다 회원 주소를 반복 저장하면 주소 변경 때 여러 행을 수정해야 하므로, 회원 정보와 주문 정보를 분리합니다.

## SQL Injection

사용자 입력을 SQL 문자열에 이어 붙이면 공격자가 SQL 구문을 바꿀 수 있습니다. PreparedStatement, JPA 파라미터 바인딩처럼 값과 SQL 구조를 분리하는 방식을 사용해야 합니다.

```java
// 안전하지 않은 방식: 문자열 결합 금지
String sql = "SELECT * FROM member WHERE email = ?";
PreparedStatement statement = connection.prepareStatement(sql);
statement.setString(1, email);
```

## 확인 질문

**SQL Injection을 막기 위해 PreparedStatement를 사용하는 이유를 설명해 주세요.**  
PreparedStatement는 SQL 구조와 사용자 입력값을 분리해 DB가 입력값을 데이터로 처리하게 합니다. 문자열 결합으로 쿼리를 만들면 입력값이 SQL 문법으로 해석될 수 있어 공격자가 조건을 변경할 위험이 있습니다.
