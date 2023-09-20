# 10. 객체지향 쿼리 언어 1

_\*분량이 많아 임의로 1, 2로 나누어 정리함_

JPA는 복잡한 검색 조건을 사용해 엔티티 객체를 조회하는 다양한 쿼리 기술을 지원한다.

JPQL은 가장 중요한 객체지향 쿼리 언어이다. Criteria, QueryDSL은 JPQL을 편리하게 사용하도록 도우는 기술이다.

## 객체지향 쿼리 소개

#### 연관된 엔티티 조회

연관된 엔티티들을 찾는 가장 단순한 검색 방법 -> 식별자로 엔티티 하나를 조회 `EntityManager.find()` -> 조회한 엔티티로 객체 그래프 탐색

더 복잡한 검색 조건의 조회는 좀 더 현실적이고 복잡한 검색 방법이 필요하다. 메모리에 연관된 모든 데이터를 올려놓고 검색하는 것은 현실성이 없다. 결국 DB에 있는 데이터를 최대한 걸러서 검색해야 한다.

ORM을 사용하면 DB 테이블이 아닌 엔티티 객체를 대상으로 하는 검색 방법이 필요하다.

-> 이런 문제 해결 위해 JPQL이 만들어졌다!

#### JPQL 특징

- 테이블이 아닌 **객체를 대상**으로 하는 **객체 지향 쿼리**이다.
- **SQL을 추상화**해 특정 DB SQL에 의존 않는다.

SQL은 DB 테이블 대상으로 하는 데이터 중심의 쿼리, JPQL은 엔티티 객체를 대상으로 하는 객체지향 쿼리다.  
-> JPQL은 객체지향 SQL이다.

JPA는 JPQL을 분석한 다음 적절한 SQL을 만들어 데이터베이스를 조회한다. 그리고 조회 결과로 엔티티 객체를 생성해 반환한다.

#### JPA 공식 지원 기능

- **JPQL(Java Persistence Query Language)** _가장 중요_
- Criteria 쿼리: JPQL 편하게 작성 하도록 도와주는 API, 빌더 클래스 모음
- 네이티브 SQL: JPA에서 JPQL 대신 직접 SQL 사용 가능

#### JPA 알아둘 기능

- QueryDSL: JPQL 편하게 작성 하도룩 도우는 빌더 클래스 모음, 비표준 오픈소스 프레임워크
- JDBC 직접 사용, MyBatis 같은 SQL 매퍼 프레임 사용: 필요시 JDBC 직접 사용 가능

### JPQL 소개

**JPQL은 엔티티 객체를 조회하는 객체지향 쿼리다.** 문법은 SQL과 비슷하고 ANSI 표준 SQL이 제공하는 기능을 유사하게 지원한다.

> ANSI 표준 SQL:  
> 미국 표준 협회(American National Standards Institute)에서 표준 SQL문을 정립 시켜 놓은 것

**JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않는다.** 데이터베이스 방언(Dialect)만 변경하면 JPQL을 수정 하지 않아도 DB를 변경할 수 있다.

**JPQL은 SQL 보다 간결하다.** 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL 보다 코드가 간결하다.

#### 간단한 JPQL 사용 코드

회원 엔티티

```java
@Entity(name = "Memeber")
public class Member {

  @Column(name = "name")
  private String username;
  //...
}
```

JPQL 사용

```java
//쿼리 생성
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```

JPQL에서 `Member`는 엔티티 이름이고, `m.username`은 테이블 컬럼명이 아닌 엔티티 객체의 필드이다.

`em.createQuery(실행할 JPQL, 반환할 엔티티 클래스 타입)`을 넘겨주고 `getResultList()`를 실행하면 JPA는 JPQL을 SQL로 변환해 데이터베이스를 조회한다. 그리고 조회한 결과로 Member 엔티티를 생성해 반환한다.

### Criteria 쿼리 소개

Criteria는 JPQL을 생성하는 빌더 클래스다. 장점은 문자가 아닌 프로그래밍 코드로 JPQL을 작성할 수 있다는 점이다. 따라서 컴파일 시점에 오류를 발견할 수 있다.

#### Criteria 장점

- 컴파일 시점에 오류를 발견할 수 있다.
- IDE를 사용하면 코드 자동완성을 지원한다.
- 동적 쿼리를 작성하기 편하다.

하이버네이트 포함 몇몇 ORM 프레임워크들은 자신만의 Criteria를 지원한다. JPA는 2.0부터 Criteria를 지원한다.

#### Criteria 사용 코드

```java
//Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

//루트 클래스(조회 시작할 클래스)
Root<Member> m = query.from(Member.class);

//쿼리 생성
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
```

위 처럼 쿼리를 코드로 작성한다. `m.get("username")`에서 필드명을 문자로 작성했는데, 이 부분도 코드로 작성하려면 메타 모델(MetaModel)을 사용하면 된다.

#### 메타 모델 API

자바가 제공하는 어노테이션 프로세서(Annotaion Processor)기능을 사용하면 어노테이션을 분석해 클래스를 생성할 수 있다. JPA는 이 기능을 사용해 Member 엔티티로부터 Memeber\_ 라는 Criteria 전용 클래스를 생성하는데 이것을 메타 모델이라 한다.

#### Criteria 단점

Criteira는 쿼리를 코드로 작성하기 때문에 동적 쿼리를 작성할 때 유용하다. 장점이 많지만 코드가 복잡하고 장황해진다. 따라서 사용하기 불편하고, Criteria로 작성한 코드는 한눈에 들어오지 않는다는 단점이 있다.

### QueryDSL 소개

QueryDSL도 JPQL 빌더 역할을 한다. QueryDSL의 장점은 코드 기반이면서 단순하고 사용하기 쉽다. 작성 코드도 JPQL과 비슷해서 한눈에 들어온다. (Criteria과 비교하면 더 단순)

#### QueryDSL 사용 코드

```java
//준비
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

//쿼리, 결과조회
List<Member> members =
  query.from(member)
  .where(member.username.eq("kim"))
  .list(member);
```

QueryDSL도 어노테이션 프로세서를 사용해 쿼리 전용 클래스를 만들어야 한다. QMember는 Member 엔티티 클래스를 기반으로 생성한 QueryDSL 전용 클래스다.

### 네이티브 SQL 소개

JPQL은 SQL을 직접 사용할 수 있는 기능을 지원 하는데 이것을 네이티브 SQL이라 한다.

JPQL을 사용해도 가끔 특정 데이터베이스에 의존하는 기능을 사용해야 할 때, SQL은 지원하지만 JPQL이 지원하지 않는 기능을 사용할 때 네이티브 SQL을 사용하면 된다.  
(특정 DB 의존 기능 예: 오라클 - CONNECT BY, 특정 DB 에서만 동작하는 SQL 힌트)

네이티브 SQL의 단점은 특정 DB에 의존하는 SQL을 작성해야 한다는 것이다. -> DB를 변경하면 SQL도 수정해야 한다.

#### 네이티브 SQL 사용 코드

```java
String sql = "SELECT ID, AGE, TEAM_ID NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resulstList = em.createQuery(sql, Member.class).getResultList();
```

### JDBC 직접 사용, MyBatis 같은 SQL 매퍼 프레임 사용

드물겠지만, JDBC 커넥션에 직접 접근 하려면 JDBC 커넥션을 획득하는 API를 제공하지 않으므로 JPA 구현체가 제공하는 방법을 사용해야 한다.

하이버네이트 JDBC 획득

```java
Session session = entityManger.unwrap(Session.class);
session.doWork(new Work() {

  @Override
  public void execute(Connection connection) throws SQLException {
    //work...
  }
});
```

먼저 JPA EntityManeger에서 Session을 구하고, session의 `doWork()`를 호출한다.

**JDBC나 마이바티스를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제 플러시해야 한다.** JPA를 우회해서 DB에 접근하므로 JPA가 우회하는 SQL에 대해 인식하지 못하기 때문이다. 영속성 컨텍스트와 DB를 불일치 상태로 만들어 데이터 무결성이 훼손될 수 있다.

이런 이슈를 해결하는 방법은 JPA를 우회해 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시해 데이터베이스와 영속성 컨텍스트를 동기화하면 된다.

스프링 프레임워크 사용  
 -> JPA와 마이바티스 손쉽게 통합 가능  
 -> AOP를 적절히 활용해 JPA를 우회해 DB에 접근하는 메소드 호출할 때 마다 영속성 컨텍스트를 플러시하면 해결 가능

## JPQL

JPQL의 사용 방법에 대해 다루는 챕터.도메인 모델 예제 통해 학습~! -> _사진?_

회원이 상품을 주문하는 다대다 관계라는 걸 주의하기. Adress 임베디드 값 타입 사용해 Orders 테이블에 포함됨.

JPQL의 특징 복습

- JPQL은 객체지향 쿼리 언어다. 테이블 대상이 아닌 엔티티 객체를 대상으로 쿼리한다.
- JPQL은 SQL을 추상화해 특정 데이터베이스 SQL에 의존 않는다.
- JPQL은 결국 SQL로 변환된다.

### 기본 문법과 쿼리 API

SELECT, UPDASTE, DELETE문 사용. 엔티티 저장은 `EntytyManager.persist()`로 하므로 INSERT문은 없음.

JPQL에서 UPDATE, DELETE문은 벌크 연산이라 한다.

#### SELECT문

```SQL
SELECT m FROM Member [AS] m where m.username = 'Hello'
```

- 대소문자 구분  
  엔티티와 속성은 대소문자를 구분한다.
- 엔티티 이름  
  Member는 클래스 명이 아닌 엔티티 명이다. 클래스명을 엔티티 명으로 사용하기 추천.
- 별칭은 필수

#### TypedQuery, Query

작성한 JPQL을 실행하려면 쿼리 객체를 만들어야 한다. 반환할 타입을 명확하게 지정할 수 있으면 TypedQuery 객체를, 없으면 Query 객체를 사용한다.

```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);

List<Member> resultList = query.getResultList();
for(Member member : resultList) {...}
```

```java
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
List resultList = query.getResultList();

for(Object o : resultList) {
  Object[] result = (Object[]) o; //결과가 둘 이상이면 Object[] 반환
  ...
}
```

Query 객체를 사용한 코드는 조회 대상이 String 타입인 회원 이름과 Interger 타입인 나이이므로 조회 대상 타입이 명확하지 않다.

Query 객체는 SELECT절의 조회 대상이 둘 이상이면 Object[]를 반환하고 하나면 Object를 반환한다.  
-> Casting이 필요 없는 TypedQuery가 더 편리

#### 결과 조회

실제 쿼리를 실행해 DB를 조회하는 메소드 호출

- `query.getResultList()`
  - 결과를 List 컬렉션으로 반환
  - 결과가 없으면 빈 컬렉션 반환
- `query.getSingleResult()`
  - 결과가 정확히 1개면 사용
  - 결과가 없으면 없으면 javax.persistence.NoResultException 예외 발생
  - 결과가 1개보다 많으면 javax.persistence.NonUniqueResultException 예외 발생

### 파라미터 바인딩

JDBC는 위치 기준 파라미터 바인딩만 지원, JPQL은 이름 기준 파라미터 바인딩도 지원.

- 이름 기준 파라미터  
  파라미터를 이름으로 구분한다. 앞에 `:`를 사용한다.

  ```java
  String usernameParam = "User1";

  TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);

  //파라미터 바인딩
  query.setParameter("username", usernameParam);
  ...
  ```

- 위치 기준 파라미터  
  `?`다음 위치 값(1부터 시작)을 주면 된다.

-> 이름 기준 파라미터 바인딩 방식 사용하기

> 파라미터 바인딩 방식을 사용하면 파라미터 값이 달라도 같은 쿼리로 인식해 JPA는 JPQL을 SQL로 파싱한 결과를 재사용할 수 있다. -> 성능 향상

### 프로젝션

SELECT절에 조회할 대상을 지정하는 것을 프로젝션(projection)이라 하고 `[SELECT] {프로젝션 대상} FROM `으로 대상을 선택한다.

#### 프로젝션 대상 종류

엔티티, 엠베디드 타입, 스칼라 타입

### 엔티티 프로젝션

```SQL
//회원 조회
SELECT m FROM Mebmer m
//회원과 연관된 팀 조회
SELECT m.team FROM Mebmer m
```

둘 다 엔티티를 대상으로 프로젝션 대상으로 사용. 원하는 객체를 바로 조회한 것.(SQL은 컬럼을 나열해 조회) 이렇게 **조회한 엔티티는 영속성 컨텍스트에서 관리**된다.

#### 임베디드 타입 프로젝션

JPQL에서 임베디드 타입은 조회의 시작점이 될 수 없다는 제약이 있다. 엔티티를 통해서만 임베디드 타입을 조회할 수 있다.

임베디드 타입은 엔티티티 타입이 아닌 값 타입이다. 직접 조회한 임베디드 타입은 영속성 컨텍스트에서 관리되지 않는다.

#### 스칼라 타입 프로젝션

숫자, 문자 등 기본 데이터 타입을 스칼라 타입이라 한다. 통계 쿼리가 주로 스칼라 타입으로 조회된다.(예. AVG(...))

전체 회원 이름 조회 쿼리.

```java
List<String> username = em.createQuery("SELECT m.username FROM Member m", Member.class).getResultList();
```

DISTINCT로 중복 데이터 제거

```SQL
SELECT DISTINCT m.username FROM Member m
```

#### 여러 값 조회

프로젝션에 엔티티 대상이 아닌 여러 값을 선택하면 TypedQuery를 사용할 수 없고 Query를 사용해야 한다.

```java
List<Object[]> resultList = em.createQuery("SELECT m.username, m.age FROM Member m").getResultList();

for(Object[] row : resultList) {
  String username = (String) row[0];
  Integer age = (Integer) row[1];
}
```

스칼라 타입이 아닌 엔티티 타입도 여러 값을 함께 조회할 수 있다.

```java
List<Object[]> resultList = em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o").getResultList();

for(Object[] row : resultList) {
  Member member = (Member) row[0];
  Product product = (Product) row[1];
  int orderAmount = (Integer) row[2];
}
```

이때도 조회한 엔티티는 영속성 컨텍스트에서 관리된다.

#### NEW 명령어

실제 애플리케이션 개발시에는 **Object[]를 직접 사용하지 않고 DTO처럼 의미 있는 객체로 변환해 사용**할 것이다.

```java
List<Object[]> resultList = em.createQuery("SELECT m.username, m.age FROM Member m").getResultList();

//객체 변환 작업
List<UserDTO> userDTOs = new ArrayList<UserDTO>();

for(Object[] row : resultList) {
  UserDTO userDTO = new UserDTO((String)row[0], (Integer)row[1]);
  userDTOs.add(userDTO);
}
return userDTOs;
```

지루한 객체 변환 작업 대신 NEW 명령어를 사용하면 간단해진다.

```java
TypedQuery<UserDTO> query = em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m", UserDTO.class);

List<UserDTO> resultList = query.getResultList();
```

NEW 명령어 사용할 때 주의할 점

1. 패키지 명을 포함한 전체 클래스 명을 입력.
2. 순서와 타입이 일치하는 생성자가 필요.

### 페이징 API

JPA는 페이징을 두 API로 추상화했다.

- `setFirstResult(int startPosition)`: 조회 시작 위치(0부터 시작)
- `setMaxResults(int maxResult)`: 조회할 데이터 수

```java
TypeQuery<Member> query = em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);

query.setFirstResult(10); //11번 부터 시작
query.setMaxResults(20); // 30번 까지
query.getResultList();
```

데이터베이스마다 다른 페이징 처리를 같은 API로 처리할 수 있는 것은 데이터베이스 방언(Dialect) 덕분이다.

페이징 SQL을 더 최적화하고 싶다면 페이징 API가 아닌 네이티브 SQL을 직접 사용해야 한다.

### 집합과 정렬

집합은 집합함수와 함께 통계 정보를 구할 때 사용한다.

#### 집합 함수

| 함수    | 설명                                                                                                           |
| :------ | :------------------------------------------------------------------------------------------------------------- |
| COUNT   | 결과 수. 반환 타입: Long                                                                                       |
| MAX_MIN | 최대, 최소 값. 문자, 숫자, 날짜 등에 사용                                                                      |
| AVG     | 평균값. 숫자타입만 사용. 반환 타입: Double                                                                     |
| SUM     | 합. 숫자타입만 사용. 반환타입: 정수합 Long, 소수합: Double, BigInteger합: BigInteger, BigDecimal합: BigDecimal |

#### 집합 함수 사용 시 참고 사항

- NULL 값은 무시.
- 값이 없는데 SUM, AVG, MIN 사용하면 NULL값. COUNT는 0.
- DISTINCT를 집합 함수 안에 사용 후 집합을 구할 수 있다.
- DISTINCT를 COUNT에 사용할 때 임베디드 타입은 지원 않는다.

#### GROUP BY, HAVING

통계 데이터를 구할 때 GROUP BY는 특정 그룹끼리 묶어주고, HAVING은 GROUP BY와 함께 사용되어 그룹화한 통계 데이터 기준으로 필터링한다. _그룹바이의 조건식...해빙..._

문법

```SQL
groupby_절 ::= GROUP BY { 단일값 경로 | 별칭 }+
having_절 ::= HAVING 조건식
```

이런 쿼리들을 보통 **리포팅 쿼리**나 **통계 쿼리**라 한다. 잘 활용하면 몇 줄의 코드로 처리할 수 있다. 하지만 통계 쿼리는 보통 전체 데이터 기준으로 처리하므로 실시간 사용에는 부담이 크다. -> 통계 결과 저장 테이블을 별도로 만들고 사용자 적은 시간에 통계 쿼리 실행해 결과 보관

#### 정렬(ORDER BY)

ORDER BY는 결과를 정렬할 때 사용한다.

문법

```SQL
orderby절_ ::= ORDER BY { 상태필드 경로 | 결과 변수 { ASC | DESC } }+
```

결과 변수는 SELECT절에 나타나는 값. -> cnt

```SQL
select t.name, COUNT(m.age) as cnt
  from Member m LEFT JOIN m.team t
  GROUP BY t.name
  ORDER BY cnt
```

### JPQL 조인

JPQL 조인은 SQL과 기능은 같고 문법만 약간 다르다.

#### 내부 조인

내부 조인은 [INNER] JOIN을 사용한다.

```java
String teamName = "팀A";
String query = "SELECT m FROM Member m INNER JOIN m.team" + "WHERE t.name = :teamName";

List<Member> members = em.createQuery(query, Member.class)
  .setParameter("teamName", teamName)
  .getResultList();
```

회원과 팀을 내부조인해서 '팀A'에 소속된 회원 조회.

생성된 내부 조인 SQL

```SQL
SELECT
  M.ID AS ID,
  M.AGE AS AGE,
  M.TEAM_ID AS TEAM_ID,
  M.NAME AS NAME
FROM
  MEMBER M INNER JOIN TEAM T ON M.TEAM_ID = T.ID
WHERE
  T.NAME = ?
```

JPQL 조인의 가장 큰 특징은 **연관 필드**를 사용한다는 것이다. (`m.team`) 연관 필드는 다른 엔티티와 연관관계를 가지기 위해 사용하는 필드를 말한다.

서로 다른 타입의 두 엔티티를 조회했으므로 TypedQuery를 사용할 수 없고, Query를 사용한다.

#### 외부 조인

외부 조인 JPQL. 기능상 SQL의 외부 조인과 같다.

```java
SELECT m
FROM member m LEFT [OUTER] JOIN m.team t
```

#### 컬렉션 조인

일대다 관계나 다대다 관계처럼 컬렉션을 사용하는 곳에 조인하는 것을 컬렉션 조인이라 한다.

- [회원 -> 팀] 조인은 다대일 조인이면서 **단일 값 연관필드(m.team)** 를 사용
- [팀 -> 회원] 조인은 반대로 일대다 조인이면서 **컬렉션 값 연관 필드(m.members)** 를 사용

`SELECT t, m FROM team t LEFT JOIN t.members m`

팀과 팀이 보유한 회원목록을 컬렉션 값 연관 필드로 외부 조인.

#### 세타 조인

WHERE 절을 사용해 세타 조인을 할 수 있다. 세타 조인은 내부 조인만 지원한다. 세타 조인을 사용하면 전혀 관계없는 엔티티도 조인할 수 있다.

`Member.username`과 `Team.name`을 조인

```java
//JPQL
select count(m) from Member m, Team t
where m.username = t.name

//SQL
SELECT COUNT(M.ID)
FROM
  MEMBER M CROSS JOIN TEAM T
WHERE
  M.USERNAME = T.NAME
```

#### JOIN ON절(JPA 2.1)

_외부 조인 대상의 조건(필터링) 지정하기!_

JPA 2.1부터 조인할 때 ON절을 지원. ON절은 조인 대상을 필터링하고 조인할 수 있다. 조인 시점에 조인 대상을 필터링한다.

모든 회원을 조회하면서 회원과 연관된 팀도 조회. 이때 팀 이름은 'A'인 팀만 조회.

```JAVA
//JPQL
select m, t from Member m
left join m.team t on t.name = 'A'

//SQL
SELECT m.*, t.* FROM Member m
LEFT JOIN Team t ON m.TEAM_ID = t.id and t.name = 'A'
```

### 페치 조인

페치 fetch 조인은 SQL의 조인 종류는 아니고 JPQL에서 **성능 최적화를 위해 제공하는 기능**이다. 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능이다. `join fetch` 명령어로 사용.

JPA 표준 명세 페치 조인 문법

`페치 조인 ::= [LEFT [OUTER] | INNER] JOIN FETCH 조인경로`

#### 엔티티 페치 조인

페치 조인으로 회원 엔티티 조회 하면서 연관된 팀 엔티티도 함께 조회하는 JPQL.

`select m from Member m join fetch m.team`

일반적인 JPQL 조인과 다르게 **페치 조인은 별칭을 사용할 수 없다.** (하이버네이트는 별칭 허용)

실행된 SQL

```SQL
SELECT
  M.*, T.*
FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```

<a href='https://ifh.cc/v-VBzcwf' target='_blank'><img src='https://ifh.cc/g/VBzcwf.jpg' border='0'></a>

엔티티 페치 조인 JPQL에서 select m으로 회원만 선택 했는데 실행된 SQL을 보면 회원과 연관된 팀도 함께 조회된다.

위 그림을 보면 회원과 팀 객체가 **객체 그래프를 유지하면서 조회**된다.

페치 조인 사용 코드

```java
String jpql = "select m from Member m join fetch m.team"

List<Memeber> members = em.createQuery(jpql, Member.class).getResultList();

for (Member member : members) {
  //페치 조인으로 회원과 팀 함께 조회 해 지연 로딩 발생 안 함
  System.out.println("username = " + member.getUsername() + ", " + "teamname = " + member.getTeam().name());
}
```

출력결과

`username = 회원1, teamname = 팀A`
`username = 회원2, teamname = 팀A`
`username = 회원3, teamname = 팀B`

회원과 팀 지연로딩 설정 가정. 회원 조회 시 페치 조인으로 팀도 함께 조회 했으므로 **연관된 팀 엔티티는 프록시가 아닌 실제 엔티티**다.  
-> 연관된 팀을 사용해도 지연 로딩이 일어나지 않는다.  
-> 실제 엔티티이므로 회원 엔티티가 준영속 상태가 되어도 연관된 팀 조회할 수 있다.

#### 컬렉션 페치 조인

일대다 관계 컬렉션을 페치 조인하기. 팀 조회 하면서 연관된 회원 컬렉션도 함께 조회. _(팀 -> 회원)_

```java
select t
from Team t join fetch t.members
where t.name = '팀A'
```

```sql
SELECT
  T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = '팀A'
```

<a href='https://ifh.cc/v-0OK6x6' target='_blank'><img src='https://ifh.cc/g/0OK6x6.jpg' border='0'></a>

> 이미지 10.7,8 삽입

TEAM 테이블에서 '팀A'는 하나지만 MEMBER 테이블과 **조인하면서 결과가 증가**해 조인 결과 테이블을 보면 같은 '팀A'가 2건 조회되었다. 따라서 컬렉션 페치 조인 결과 객체에서 teams 결과 예제를 보면 주소가 0x100으로 같은 '팀A'를 2건 가지게 된다.

> 일대다 조인은 결과가 증가할 수 있지만 일대일, 다대일 조인은 결과가 증가하지 않는다.

컬렉션 페치 조인 사용

```JAVA
String jpql = "select t from Team t join fetch t.members where t.name = '팀A'"
List<Team> teams = em.createQuery(jpql, Team.class).getResultList();

for(Team team : teams) {
  System.out.println("teamname = " + team.getName() + ", team = " + team);

  for(Member member : team.getMembers()) {
    //페치 조인으로 팀과 회원을 함께 조회해 지연 로딩 발생 안 함
    System.out.println("->username = " + member.getUsername() + ", member = " + member);
  }
}
```

출력 결과

`teamname = 팀A, team = Team@0x100`
`->username = 회원1, member = Member@0x200`
`->username = 회원2, member = Member@0x300`
`teamname = 팀A, team = Team@0x100`
`->username = 회원1, member = Member@0x200`
`->username = 회원2, member = Member@0x300`

같은 '팀A'가 2건 조회됨.

#### 페치 조인과 DISTINCT

JPQL의 DISTINCT 명령어는 SQL에 DISTINCT를 추가하고, 애플리케이션에서 한 번 더 중복을 제거한다.

직전 컬렉션 페치 조인에서 DISTINCT를 사용해 팀A가 중복 조회되지 않도록 하기

```sql
select distint t
from Team t join fetch t.members
where t.name = '팀A'
```

JPQL에 DISTINCT를 사용하면 먼저 SQL에 DISTINCT가 추가된다. 하지만 각 로우의 데이터가 다르므로 효과가 없다. _('팀A, 회원1', '팀A, 회원2')_

다음으로 애플리케이션에서 distinct 명령어를 보고 중복된 데이터를 걸러낸다. 따라서 중복인 팀A는 하나만 조회된다.

#### 페치 조인과 일반 조인의 차이

페치 조인이 아닌 일반 조인만 사용하면?

내부 조인 JPQL

```java
select t
from Team t join t.members =m
where t.name = '팀A'
```

실행된 SQL

```SQL
SELECT
  T.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME='팀A'
```

JPQL에서 팀과 멤버를 조인했지만 실행된 SQL을 보면 회원 컬렉션이 함께 조회되지 않고 팀만 조회 되었다. 조인했던 회원은 전혀 조회되지 않는다...!

**=> JPQL은 결과를 반환할 때 연관관계까지 고려하지 않는다. 단지 SELECT절에 지정한 엔티티만 조회할 뿐이다.**

만약 회원 컬렉션을 지연 로딩으로 설정하면 프록시나 아직 초기화하지 않은 컬렉션 래퍼를 반환한다. 즉시 로딩이면 회원 컬렉션을 즉시 로딩하기 위해 쿼리를 한 번 더 실행한다.

반면 페치 조인을 사용하면 연관된 엔티티도 함께 조회한다.

#### 페치 조인의 특징과 한계

페치 조인은 SQL 한 번으로 연관된 엔티티도 함께 조회해 SQL 호출 횟수를 줄여 성능을 최적화할 수 있다.

엔티티에 직접 적용하는 로딩 전략은 애플리케이션 전체에 영향을 미치므로 글로벌 로딩 전략이라 부른다. 페치 조인은 글로벌 로딩 전략보다 우선한다.

최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하면 애플리케이션 전체에서 항상 즉시 로딩이 일어난다.  
-> 전체로 보면 사용하지 않는 엔티티를 자주 로딩해 성능에 악영향  
**=> 글로벌 로딩 전략은 될 수 있으면 지연 로딩, 최적화가 필요하면 페치 조인을 적용하는 것이 효과적**

페치 조인 사용하면 연관된 엔티티를 쿼리 시점에 조회  
-> 지연 로딩 발생X  
-> **준영속 상태에서도 객체 그래프 탐색 가능**

#### 페치 조인 한계

- 페치 조인 대상에는 별칭을 줄 수 없다
- 둘 이상의 컬렉션을 페치할 수 없다
- 컬렉션을 페치 조인하면 페이징 API(`setFirstResult`, `setMaxResults`)를 사용할 수 없다.
  - 컬렉션(일대다)가 아닌 단일 값 연관 필드(일대일, 다대일)들은 페치 조인을 사용해도 페이징 API 사용 가능
  - 하이버네이트는 컬렉션을 페치 조인하고 페이징 API를 사용하면 경고 로그를 남기면서 메모리에서 페이징처리 한다. 데이터가 많으면 성능 이슈와 메모리 초과 예외가 발생할 수 있어 위험하다.

페치 조인은 성능 최적화에 유용하다. 그리고 실무에서 자주 사용하게 된다. 하지만 모든 것을 페치 조인으로 해결할 수는 없다. **페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다.** 반면 여러 테이블을 조인해 전혀 다른 결과를 내야 한다면 여러 테이블에 필요한 필드들만 조회해 DTO로 반환하는 것이 더 효과적이다.

### 경로 표현식

JPQL에서 사용하는 경로 표현식(Path Expression)을 알아보고 경로 표현식을 통한 묵시적 조인도 알아보자. 경로 표현식은 `.`을 찍어 객체 그래프를 탐색하는 것이다.

#### 경로 표현식의 용어 정리

- **상태 필드(state field)**: 단순히 값을 저장하기 위한 필드(필드 or 프로퍼티)
  - 예: `t.username`, `t.age`
- **연관 필드(association field)**: 연관관계를 위한 필드, 임베디드 타입(필드 or 프로퍼티)
  - **단일 값 연관필드**: 대상이 엔티티(`@ManyToOne`, `@OneToOne`)
    - 예: `m.team`
  - **컬렉션 값 연관필드**: 대상이 컬렉션(`@OneToMany`, `@ManyToMany`)
    - 예: `m.orders`

#### 경로 표현식 특징

JPQL에서 경로 표현식을 사용해 경로 탐색을 하려면 3가지 경로의 특징을 이해해야 한다.

- 상태 필드 경로: 경로 탐색의 끝
- 단일 값 연관 경로: 묵시적 내부 조인 발생. 계속 탐색할 수 있다.
- 컬렉션 값 연관 경로: 묵시적 내부 조인 발생. 더 탐색할 수 없다. 단, FROM절에서 조인 통해 별칭을 얻으면 별칭으로 탐색할 수 있다.

묵시적 조인은 모두 내부 조인이다. 외부 조인은 명시적으로 JOIN 키워드를 사용해야 한다.

- 명시적 조인: JOIN을 직접 적어주는 것.
- 묵시적 조인: 경로 표현식에 의해 묵시적 조인이 일어나는 것. 내부 조인만 할 수 있다.

임베디드 타입에 접근하는 것도 단일 값 연관 경로 탐색이지만 이미 상위 테이블에 포함되어 있으므로 조인이 발생하지 않는다.

컬렉션 값 연관 경로 탐색 시 컬렉션 값에서 경로 탐색을 시도하지 않도록 주의하기...!

컬렉션까지는 경로 탐색이 가능하지만 컬렉션에서 경로 탐색을 시작하는 것(`t.members.username`)은 안 된다. -> 조인을 사용해 새로운 별칭을 획득하면 컬렉션의 별칭부터 다시 경로 탐색 가능.

#### 경로 탐색을 사용한 묵시적 조인 시 주의사항

경로 탐색을 사용하면 묵시적 조인이 발생해 SQL에서 내부 조인이 일어날 수 있다.

- 항상 내부 조인이다.
- 컬렉션은 경로 탐색의 끝이다.
- 경로 탐색은 주로 SELECT, WHERE 절(다른 곳에서도 사용됨)에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM 절에 영향을 준다.

묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다는 단점이 있다. -> 성능이 중요하면 분석하기 쉽도록 명시적 조인 사용하자.

### 서브 쿼리

JPQL도 SQL처럼 서브 쿼리를 지원하는데 몇 가지 제약이 있다. **서브쿼리를 WHERE, HAVING절에서만 사용할 수 있다.**

#### 서브 쿼리 함수

서브쿼리는 다음 함수들과 같이 사용 가능.

- [NOT] EXISTS (subquery)
  - 서브쿼리에 결과가 존재하면 참
- {ALL | ANY | SOME} (subquery)
  - 비교 연산자와 같이 사용
  - ALL: 조건 모두 만족시 참
  - ANY, SOME: 조건 하나라도 만족하면 참
- [NOT] IN (subquery)
  - 서브쿼리 결과 중 하나라도 같은 것이 있으면 참

### 조건식

#### 타입 표현

| 종류        | 설명                                                                                    | 예제                                                                                                       |
| :---------- | :-------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| 문자        | 작은 따옴표 사이 표현<br> 작은 따옴표 표현시 연속 두 개 ('') 사용                       | 'HELLO'<br> 'She''s'                                                                                       |
| 숫자        | L(Long 타입 지정)<br> D(Double 타입 지정)<br> F(Float 타입 지정)                        | 10L<br> 10D<br> 10F                                                                                        |
| 날짜        | DATE {d 'yyyy-mm-dd'}<br> TIME {t 'hh-mm-ss'}<br> DATETIME {ts 'yyyy-mm-dd hh:mm:ss.f'} | {d '2012-03-24'}<br> {t '10-11-11'}<br> {ts '2012-03-24 10-11-11.123'}<br> m.createDate = {d '2012-03-24'} |
| Boolean     | TRUE, FALSE                                                                             |                                                                                                            |
| Enum        | 패키지명을 포함한 전체 이름 사용                                                        | jpabook.MemberType.Admin                                                                                   |
| 엔티티 타입 | 엔티티 타입을 표현. 주로 상속과 관련해서 사용.                                          | Type(m) = Member                                                                                           |

#### 연산자 우선 순위

1. 경로 탐색 연산(.)
2. 수학 연산: +, -(단항 연산자), \*, \/, +, -
3. 비교 연산: =, >, >=, <, <=, <>, [NOT] BETWEEN, [NOT] LIKE, [NOT] IN, IS [NOT] NULL, IS [NOT] EMPTY, [NOT] MEMBER [OF], [NOT] EXISTS
4. 논리 연산: NOT, AND, OR

- LIKE 식  
  문법: 문자표현식 [NOT] LIKE 패턴값 [ESCAPE 이스케이프문자]  
  설명: 문자표현식과 패턴값 비교
  - %: 값이 없거나, 아무 값 입력되어도 됨
  - \_: 한 글자 아무 값 입력, 값이 있어야 함

#### 컬렉션 식

컬렉션 식은 컬렉션에만 사용하는 특별한 식이다. 컬렉션에는 컬렉션 식만 사용 가능!

- 빈 컬렉션 비교 식  
  문법: {컬렉션 값 연관 경로} IS [NOT] EMPTY  
  설명: 컬렉션 값이 비었으면 참
- 컬렉션의 멤버 식  
  문법: {엔티티나 값} [NOT] MEMBER [OF] {컬렉션 값 연관 경로}  
  설명: 엔티티나 값이 컬렉션에 포함되어 있으면 참

#### 스칼라 식

스칼라는 숫자, 문자, 날짜, CASE, 엔티티 타입 같은 가장 기본적 타입들을 말한다. 스칼라에 사용하는 식.

- **수학 식**
  - +, -: 단항 연산자
  - \*, \/, +, -: 사칙연산
- **문자함수**  
   |함수|설명|예제|
  |:---|:---|:---|
  |CONCAT(문자1, 문자2, ...)|문자를 합한다.|CONCAT('A','B')=AB|
  |SUBSTRING(문자, 위치, [길이])|위치부터 시작해 길이만큼 문자를 구한다. 길이가 없으면 나머지 전체 길이 의미.|SUBSTRING('ABCDEF', 2, 3)=BCD|
  |TRIM\([[LEADING \| TRAILING \| BOTH] [트림문자] FROM] 문자)|LEADING: 왼쪽만<BR> TRAINING: 오른쪽만<BR> BOTH: 양쪽 다<BR> 트림 문자를 제거. 기본값은 공백이다.|TRIM(' ABC ')='ABC'|
  |LOWER(문자)|소문자로 변경|LOWER('ABC')='abc'|
  |UPPER(문자)|대문자로 변경|UPPER('abc')='ABC'|
  |LENGTH(문자)|문자 길이|LENGTH('ABC')=3|
  |LOCATE(찾을 문자, 원본 문자, [검색 시작 위치])|검색위치부터 문자를 검색. 1부터 시작, 못 찾으면 0 반환|LOCATE('DE', 'ABCDEFG')=4|
- **수학 함수**  
  |함수|설명|예제|
  |:---|:---|:---|
  |ABS(수학식)|절대값을 구한다.|ABS(-10)=10|
  |SQRT(수학식)|제곱근|SQRT(4)=2.0|
  |MOD(수학식, 나눌 수)|나머지|MOD(4,3)=1|
  |SIZE(컬렉션 값 연관 경로식)|컬렉션의 크기를 구한다.|SIZE(t.members)|
  |INDEX(별칭)|LIST 타입 컬렉션의 위치값을 구함. 단 컬렉션이 `@OrderColumn`을 사용하는 LIST 타입일때만 사용.|t.members m where INDEX(m)>3|
- **날짜 함수**  
  DB의 현재 시간을 조회한다.
  - CURRENT_DATE: 현재 날짜
  - CURRENT_TIME: 현재 시간
  - CURRENT_TIMESTAMP: 현재 날짜 시간

DB들은 각자의 방식으로 더 많은 날짜 함수를 지원한다. 각각의 날짜 함수는 하이버네이트가 제공하는 데이터베이스 방언에 등록되어 있다.

#### CASE 식

특정 조건에 따라 분기할 때 CASE식을 사용한다. 4가지 종류

- 기본 CASE
- 심플 CASE
- COALESCE
- NULLIF

#### CASE 식- 기본 CASE

문법

```JAVA
CASE
  {WHEN <조건식> THEN <스칼라식>}
  ELSE <스칼라식>
END
```

예

```JAVA
select
  case when m.age <= 10 then '학생요금'
       when m.age >= 60 then '경로요금'
       else '일반요금'
  end
from Member m
```

#### CASE 식 - 심플 CASE

심플 CASE는 조건식 사용X. 자바의 switch case문과 비슷하다.

문법

```java
CASE <조건대상>
  {WHEN <스칼라식1> THEN <스칼라식2>}+
  ELSE <스칼라식>
END
```

예

```JAVA
select
  case t.name
    when '팀A' then '인센티브110%'
    when '팀B' then '인센티브120%'
    else '인센티브105%'
  end
from Team t
```

#### CASE 식 - COALESCE

문법: COALESCE(<스칼라식> {,<스칼라식>}+)  
설명: **스칼라식을 차례대로 조회해 NULL이 아니면 반환한다.**

예
`select coalesce(m.username, '이름 없는 회원') from Member m`  
-> m.username이 null이면 '이름 없는 회원'을 반환

#### CASE 식 - NULLIF

문법: NULLIF(<스칼라식>, <스칼라식>)
설명: 두 값이 같으면 NULL을 반환하고, 다르면 첫 번째 값을 반환한다. 집합 함수는 NULL을 포함하지 않으므로 보통 집합함수와 함께 사용한다.

예
`select NULLIF(m.username, '관리자') from Member m`  
-> 사용자 이름이 '관리자'면 null을, 나머지는 본인 이름 반환.

### 다형성 쿼리

JPQL로 부모 엔티티를 조회하면 그 자식 엔티티도 함께 조회된다. _(예제: abstract class Item의 자식 Book, Album, Movie... Item에는 `@Inheritance`로 SINGLE_TALBE 전략 지정...)_

JPQL에서 `select i from Item i`로 조회하면 자식도 함께 조회된다.

단일 테이블 전략(InheritanceType.SINGLE_TALBE)을 사용할 때 실행되는 SQL은 `SELECT * FROM ITEM`.

조인 전략(InheritanceType.JOINED)를 사용할 때 실행되는 SQL

```SQL
SELECT
  i.ITEM_ID, i.DTYPE, i.name, i.price, i.stockQuantity,
  b.author, b.isbn,
  a.artist, a.ect,
  m.actor, m.director
FROM
  Item i
left outer join
  Book b on i.ITEAM_ID=b.ITEM_ID
left outer join
  Album a on i.ITEAM_ID=a.ITEM_ID
left outer join
  Movie m on i.ITEAM_ID=m.ITEM_ID
```

#### TYPE

TYPE은 엔티티의 상속 구조에서 **조회 대상을 특정 자식 타입으로 한정할 때 주로 사용**한다.

TYPE 사용 JPQL  
`select i from Item i where type(i) IN(Book, Movie)`  
-> Item 중 Book, Movie 조회

실행 SQL  
`SELECT i FROM Item i WHERE i.DTYPE in ('B', 'M')`

#### TREAT(JPA 2.1)

TREAT는 자바의 타입 캐스팅과 비슷하다. 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다. JPA 표준은 FROM, WHERE절에서 사용, 하이버네이트는 SELECT절에서도 사용 가능 하다.

TREAT 사용 JPQL  
`select i from Item i where treat(i as Book).author = 'kim'`

실행 SQL  
`SELECT i.* FROM Item i WHERE i.DTYPE ='B' and i.author='kim'`

JPQL을 보면 treat를 사용해 부모 타입인 Item을 자식 타입인 Book으로 다룬다. _부모 타입을 자식 타입으로 다루어 자식의 필드에 접근한다._

### 사용자 정의 함수 호출(JPA 2.1)

문법:  
`function_invocation::= FUNCTION(function_name {, function_arg}*)`

예:  
`select function('group_concat', i.name) from Item i`

하이버네이트를 사용하면 방언 클래스를 상속해 구현하고, 사용할 데이터베이스 함수를 미리 등록해야 한다.

방언 클래스 상속

```java
public class MyH2Dialect extends H2Dialect {

  public MyH2Dialect() {
    registerFunction("group_concat", new StrandardSQLFunction("group_concat", StandardBasicTypes.STRING));
  }
}
```

persistence.xml의 hibernate.dialect에 해당 방언 등록

```xml
<proterty name="hibernate.dialect" value="hello.MyH2Dialect" />
```

하이버네이트 구현체 사용한 사용자 정의 함수 사용  
`select group_concat(i.name) from Item i`

### 기타 정리

- enum은 = 비교 연산자만 지원한다
- 임베디드 타입은 비교를 지원하지 않는다

#### EMPTY STRING

JPA 표준은 ''을 길이 0인 Empty String으로 정함. DB에 따라 null로 사용하는 DB도 있다.

#### NULL 정의

- 조건을 만족하는 데이터가 하나도 없으면 NULL이다.
- NULL은 알 수 없는 값이다.(unknown value) NULL과의 모든 수학적 계산 결과는 NULL이다.
- NULL == NULL은 알 수 없는 값.
- NULL IS NULL은 참

JPA 표준 명세 NULL(U), TRUE, FALSE의 논리 계산 정의

- AND 연산  
  |AND|T |F |U |
  |:--|:--|:--|:--|
  |T|T|F|U|
  |F|F|F|F|
  |U|U|F|U|
- OR 연산  
  |OR|T|F|U|
  |:--|:--|:--|:--|
  |T|T|T|T|
  |F|T|F|U|
  |U|T|U|U|
- NOT 연산  
  |NOT||
  |:--|:--|
  |T|F|
  |F|T|
  |U|U|

### 엔티티 직접 사용

#### 기본 키 값

객체 인스턴스는 참조 값으로 식별하고 테이블 로우는 기본 키 값으로 식별한다. JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키 값을 사용한다.

`select count(m.id) from Member m` -> 엔티티 아이디 사용  
`select count(m) from Member m` -> 엔티티 직접 사용

**엔티티를 직접 사용하면 JPQL이 SQL로 변환될 때 해당 엔티티의 기본 키를 사용**한다. 따라서 실제 실행된 SQL은 둘 다 같다.

엔티티를 파라미터로 직접 받는 코드

```JAVA
String qlString = "select m from Member m where m = :member";
List resultList = em.createQuery(qlString)
  .setParameter("member", member)
  .getResultList();
```

위 코드로 실행된 sql을 보면 엔티티를 직접 사용하는 부분(`where m = :member`)에서 기본 키 값을 사용하도록 변환된다. (`where m.id =?`)

#### 외래 키 값

외래 키를 사용하는 예. 특정 팀에 소속된 회원 찾기.

외래 키 대신 엔티티 직접 사용하는 코드

```java
Team team = em.find(Team.class, 1L);

String qlString = "select m from Member m where m.team = :team";
List resultList = em.createQuery(qlString)
  .setParameter("team", team)
  .getResultList();
```

기본 키 값이 1L인 팀 엔티티를 파라미터로 사용. m.team은 team_id 라는 외래키와 매핑되어 있다.

실행되는 sql

```sql
select m.*
from Member m
where m.team_id=?
```

m.team.id를 보면 Member와 Team 간에 묵시적 조인 발생X -> Member 테이블이 team_id 외래키를 가지고 있으므로 묵시적 조인은 발생X

m.team을 사용하든 m.team.id를 사용하든 생성되는 sql은 같다.

### Named 쿼리: 정적 쿼리

JPQL 쿼리는 크게 동적 쿼리와 정적 쿼리로 나뉜다.

- **동적 쿼리**  
  `em.createQuery("select ...")`처럼 JPQL을 문자로 완성해 직접 넘기는 것. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성할 수 있다.
- **정적 쿼리**  
  미리 정의한 쿼리에 이름을 부여해 필요할 때 사용할 수 있다. 이것을 Named 쿼리라 한다. 한 번 정의하면 변경할 수 없는 정적인 쿼리다.

Named 쿼리는 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해둔다. -> 오류 빨리 확인, 사용 시점에 파싱된 결과를 재사용 -> 성능상 이점

Named 쿼리는 정적 sql이 생성됨 -> DB 조회 성능 최적화에 도움

Named 쿼리는 `@NamedQuery` 어노테이션을 사용해 자바 코드에 작성하거나 XML 문서에 작성할 수 있다.

#### Named 쿼리를 어노테이션에 정의

```java
@Entity
@NamedQuery(
  name = "Member.findByUsername",
  query = "select m from Member m where m.username = :username")
public class Member {...}
```

`@NamedQuery.name`에 쿼리 이름 부여, `@NamedQuery.query`에 사용할 쿼리 입력.

`@NamedQuery` 사용

```java
List<Member> resultList = em.createQuery("Member.findByUsername", Member.class)
  .setParameter("username", "회원1")
  .getResultList();
```

> Named 쿼리를 사용할 때 앞에 엔티티 이름을 주었는데, Named 쿼리는 영속성 유닛 단위로 관리되므로 충돌 방지를 위해 엔티티 이름을 앞에 주었다. 관리도 쉬워진다.

#### Named 쿼리를 XML에 정의

JPA에서 어노테이션으로 작성할 수 있는 것은 XML로도 작성할 수 있다. **Named 쿼리를 작성할 때는 어노테이션 보다 XML을 사용하는 것이 더 편리하다...!** (자바에서 멀티라인 문자 다루기 힘들기 때문에...)

XML에 정의한 Named 쿼리(META-INF/ormMember.xml)

```xml
<?xml version="1.0" encoding="UTF-8">
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">

  <named-query name="Member.findByUsername">
    <query><CDATA[
      select m
      from Member m
      where m.username = :username
    ]></query>
  </named-query>

  <named-query name="Member.count">
    <query>select count(m) from Member m</query>
  </named-query>

</entity-mappings>
```

> XML에서 &, <, >는 예약문자다. 대신 \&amp;, \&lit;, \&gt;를 사용. \<![CDATA[ ]]>를 사용 하면 그 사이 문장을 그대로 출력.(예약문자도 사용 가능)

정의한 ormMember.xml을 인식하도록 META-INF/persistence.xml에 다음 코드를 추가.

```xml
<persistence-unit name="jpabook">
  <mapping-file>META-INF/ormMember.xml</mapping-file>
  ...
```

> META-INF/orm.xml은 JPA가 기본 매핑파일로 인식해서 별도의 설정을 하지 않아도 된다.

#### 환경에 따른 설정

만약 XML과 어노테이션에 같은 설정이 있으면 XML이 우선권을 가진다. 따라서 애플리케이션이 운영 환경에 따라 다른 쿼리를 실행해야 한다면 각 환경에 맞춘 XML을 준비해 두고 XML만 변경해서 배포하면 된다.
