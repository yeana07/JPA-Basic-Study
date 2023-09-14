# 10. 객체지향 쿼리 언어 1

_\*분량이 많아 임의로 1, 2로 나누어 정리함_

\* JPQL 소개에 ANSI 표준 SQL 정보 찾아 쓰기~~~~~~~~~~~~~~~!

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

> ANSI 표준 SQL?

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
List<Member> resultList = em.createQuery(cq).getResultList;
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
List<Member> resulstList = em.createQuery(sql, Member.class).getResultList;
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

### 기본 문법과 쿼리 API

### 파라미터 바인딩

### 프로젝션

### 페이징 API

### 집합과 정렬

### JPQL 조인

### 페치 조인

### 경로 표현식

### 서브 쿼리

### 조건식

### 다형성 쿼리

### 사용자 정의 함수 호출(JPA 2.1)

### 기타 정리

### 엔티티 직접 사용

### Named 쿼리: 정적 쿼리

###
