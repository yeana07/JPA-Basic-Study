# 10. 객체지향 쿼리 언어 2

_\*분량이 많아 임의로 1, 2로 나누어 정리함_

_\*그래도...내용이...많다.🙄 JPQL 사용 문법을 보려면 코드로 보는 게 더 파악하기 좋아서, 코드예제 전체를 그대로 옮겨 적어 분량이 더 많음._

## Criteria

Criteria는 JPQL을 자바 코드로 작성하도록 도우는 빌더 클래스 API이다.

**장점:** 문법 오류를 컴파일 단계에서 잡을 수 있고, 문자 기반 JPQL 보다 동적 쿼리를 안전하게 생성할 수 있다.

**단점:** 코드가 복잡해 직관적인 이해가 힘들다.

### Criteria 기초

Criteria API는 javax.psersistence.criteria 패키지에 있다.

모든 회원 엔티티를 조회하는 가장 단순한 Criteria 쿼리

```java
//JPQL: select m from Member m (Criteria 결과로 생성된 JPQL)

CriteriaBuilder cb = em.getCriteiraBuilder(); //Criteria 쿼리 빌더 - 1

//Criteria 생성, 반환 타입 지정 - 2
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class); //FROM절 - 3
cq.select(m); //SELECT절 - 4

TypedQuery<Member> query = em.cretaeQuery(cq);
List<Member> members = query.getResultList();
```

1. Criteria 쿼리를 생성하려면 먼저 Criteria 빌더를 얻기. EntityManager나 EntityManagerFactory에서 얻는다.
2. Criteria 쿼리 빌더에서 Criteria 쿼리(CriteriaQuery)를 생성. 이때 반환 타입을 지정할 수 있음.
3. FORM절 생성. 반환된 m은 Criteria에서 사용하는 특별한 별칭. m을 **조회의 시작**이라는 의미로 **쿼리 루트(Root)** 라고 한다.
4. SELECT절 생성

Criteria 쿼리를 완성하면 `em.createQuery(cq)`에 완성된 Criteria 쿼리를 넣어주면 된다.

검색 조건과 정렬을 추가한 Criteria 쿼리

```java
//JPQL
//select m from Member m
//where m.username='회원1'
//order by m.age desc

CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class); //FROM절 생성

//검색 조건 정의
Predicate usernameEqual = cb.equal(m.get("username"), "회원1");

//정렬 조건 정의
javax.persistence.criteria.Order ageDesc = cb.desc(m.get("age"));

//쿼리 생성
cq.select(m)
  .where(usernameEqual) //WHERE절 생성
  .orderBy(ageDesc); //ORDER BY절 생성

List<Member> resultList = em.createQuery(cq).getResultlist();
```

- 쿼리 루트 Query Root와 별칭

  - `Root<Member> m = cq.from(Member.class)`에서 `m`이 쿼리루트
  - 쿼리 루트는 조회의 시작점
  - Criteria에서 사용되는 특별한 별칭이다
  - 별칭은 엔티티에만 부여

- Criteria의 경로 표현식
  - `m.get("username")`는 JPQL의 `m.username`과 같다
  - `m.get("team").get("name")`은 JPQL의 m.team.name과 같다

숫자 타입 검색하는 Criteria 쿼리

```java
//select m from Member m
//where m.age > 10 order by m.age desc

Root<Member> m = cq.from(Member.class);

//타입 정보 필요
Predicate ageGt = cb.greaterThan(m.<Integer>get("age"), 10);

cq.select(m);
cq.where(ageGt);
cq.orderBy(cb.desc(m.get("age")));
```

`cb.greaterThan(m.<Integer>get("age"), 10)`을 보면 제네릭으로 타입 정보를 주고 있다. `m.get("age")`에서는 "age"의 타입 정보를 알지 못한다. 따라서 **제네릭으로 반환 타입 정보를 명시**해야 한다.(보통 String 같은 문자 타입은 지정X)

### Criteria 쿼리 생성

Criteria를 사용하려면 `CriteriaBuilder.createQuery()` 메소드로 Criteria 쿼리(CriteriaQuery)를 생성하면 된다.

CriteriaBuilder

```java
public interface CriteriaBuilder {

  CriteriaQuery<Object> createQuery(); //조회값 반환 타입: Object

  //조회값 반환 타입: 엔티티, 임베디드 타입, 기타
  <T> CriteriaQuery<T> createQuery(Class<T> resultClass);

  CriteriaQuery<Tuple> createTupleQuery(); //조회값 반환 타입: Tuple

  ...
}
```

CriteriaBuilder 인터페이스를 보면 Criteria 쿼리를 생성할 때 파라미터로 쿼리 결과에 대한 반환 타입을 지정할 수 있다.

반환 타입이 둘 이상이거나 지정할 수 없으면 타입을 지정하지 않고 Object로 반환 받으면 된다.

물론 반환 타입이 둘 이상이면 Object[]를 사용하는 것이 편리하다.  
`CriteriaQuery<Object[]> cq = cb.creatQuery(Obejct[].class)`

### 조회

SELECT절을 만드는 `select()`.

CriteriaQuery

```java
public interface CriteriaQuery<T> extends AbstractQuery<T> {

  //한 건 지정
  CriteriaQuery<T> select(Selection<? extends T> selection);

  //여러 건 지정
  CriteriaQuery<T> multiselect(Selection<?> ... selections);

  //여러 건 지정
  CriteriaQuery<T> multiselect(List<Selection<?>> selectionList);

  ...
}
```

#### 조회 대상을 한 건, 여러 건 지정

select에 조회 대상 하나만 지정

`cq.select(m)` `//JPQL: select m`

조회 대상 여러 건 지정 -> multiselect 사용

`//JPQL: select m.username, m.age`  
`cq.multiselect(m.get("username"), m.get("age"))`

여러 건 지정은 `cb.array`도 사용 가능

`cq.select(cb.array(m.get("username"), m.get("age")))`

#### DISTINCT

select, multiselect 다음에 `distinct(true)`를 사용하면 된다.

`//JPQL: select distinct m.username, m.age`  
`cq.multiselect(m.get("username", m.get("age")).distinct(true))`

완성된 코드

```java
//JPQL: select distinct m.username, m.age from Member m

CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class);
cq.multiselect(m.get("username"), m.get("age")).distinct(true);

TypedQuery<Obejct[]> query = em.createQuery(cq);
List<Object[]> resultList = query.getResultList();
```

#### NEW, construct()

JPQL에서 `select new 생성자()` 구문을 Criteria에서는 `cb.construct(클래스 타입, ...)`으로 사용.

`<Y> CompoundSelection<Y> construct(Class<T> resultClass, Selection<?> ... selections)`

`construct()` 실제 사용 코드

```java
//JPQL: select new jpabook.domain.MemberDTO(m.username, m.age)
//from Member m

CriteriaQury<MebmerDTO> cq = cb.createQuery(MemberDTO.class);
Root<Member> m = cq.from(Member.class);

cq.select(cb.construct(MemberDTO.class, m.get("username"), m.get("age")));

TypedQuery<MemberDTO> query = em.createQuery(cq);
List<MemberDTO> resultList = qeury.getResultList();
```

JPQL에서는 패키지명을 다 적었지만, Criteria는 코드를 직접 다루므로 간략하게 사용(`MemberDTO.class`).

#### 튜플

Criteria는 Map과 비슷한 튜플이라는 특별한 반환 객체를 제공한다. _조회 대상이 둘 이상일 때 유용_

튜플로 엔티티도 조회할 수 있다.

```java
//JPQL: select m.username, m.age from Member m

CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQury<Tuple> cq = cb.createTupleQuery();

Root<Member> m = cq.from(Member.class);

cq.multiselect(
  m.get("username").alias("username"), //튜플에서 사용할 튜플 별칭 - 1
  m.get("age").alias("age")
);

TypedQuery<Tuple> query = em.createQuery(cq);
List<Tuple> resultList = qeury.getResultList();
for (Tuple tuple : resultList) {
  //튜플 별칭으로 조회 - 2
  String username = tuple.get("username", String.class);

  Integer age = tuple.get("age", Integer.class);
}
```

튜플을 사용하려면 `cb.createTupleQuery()` 또는 `cb.createQuery(Tuple.class)`로 Criteria를 생성한다.

1. 튜플은 튜플의 검색 키로 사용할 **튜플 전용 별칭을 필수로 할당.** 별칭을 `alias()`로 지정
2. 선언해둔 튜플 별칭으로 데이터 조회

튜플은 이름 기반이므로 순서 기반 Object[] 보다 안전하다. `tuple.getElements()`로 현재 튜플의 별칭과 자바 타입도 조회 가능.

> 튜플은 Map과 비슷한 구조여서 별칭을 키로 사용한다.

### 집합

#### GROUP BY

팀 이름별 나이가 가장 많은 사람과 가장 적은 사람 구하기.

```java
/*
  JPQL:
  select m.team.name, max(m.age), min(m.age)
  from Member m
  group by m.team.name
*/

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQury<Object[]> cq = cb.createQuery(Object[].class);
Root<Member> m = cq.from(Member.class);

Expression maxAge = cb.max(m.<Integer>get("age"));
Expression minAge = cb.min(m.<Integer>get("age"));

cq.multiselect(m.get("team").get("name"), maxAge, minAge);
cq.groupBy(m.get("team").get("name")); //GROUP BY

TypedQuery<Object[]> query = em.createQuery(cq);
List<Object[]> resultList = qeury.getResultList();
```

#### HAVING

위 조건에 팀에 가장 나이 어린 사람이 10살을 초과하는 팀을 조회한다는 조건을 추가.

```java
cq.multiselect(m.get("team").get("name"), maxAge, minAge)
  .groupBy(m.get("team").get("name"))
  .having(cb.gt(minAge, 10));
```

### 정렬

정렬 조건도 Criteria 빌더를 통해 생성.

`cb.desc(...)` 또는 `cb.asc(...)`로 생성한다.

### 조인

조인은 `join()` 메소드와 `JoinType` 클래스를 사용.

```java
public enum JoinType {

  INNER, //내부 조인
  LEFT,  //왼쪽 외부 조인
  RIGTH  //오른쪽 외부 조인
         //JPA 구현체나 DB에 따라 지원하지 않을수도 있다.
}
```

JOIN 예

```java
/*
  JPQL:
  select m, t from Member m
  inner join m.team t
  where t.name = '팀A'
*/

Root<Member> m = cq.from(Member.class);
Join<Member, Team> t = m.join("team", JoinType.INNER);

cq.multiselect(m, t)
  .where(cb.equal(t.get("name"), "팀A"));
```

쿼리 루트(m)에서 회원과 팀을 조인하고, 조인한 팀에 t 라는 별칭을 주었다. 조인 타입을 생략하면 내부 조인을 사용한다. 외부 조인은 `JoinType.LEFT`로 설정.

FETCH JOIN은 다음과 같이 사용.

`m.fetch("team", JoinType.LEFT)`

### 서브 쿼리

#### 간단한 서브 쿼리

메인 쿼리와 서브 쿼리 간 관련이 없는 단순한 서브 쿼리.

평균 나이 이상의 회원을 구하는 서브 쿼리.

```java
/*
  JPQL:
    select m from Member m
    where m.age >=
      (select AVG(m2.age) from Member m2)
*/

CriteriaBuilder cb = em.getCriteriaBuiler();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);

//서브 쿼리 생성 - 1
SubQeury<Double> subQuery = mainQuery.subquery(Double.class);
Root<Member> m2 = subQuery.from(Member.class);
subQuery.select(cb.avg(m2.<Integer>get("age")));

//메인 쿼리 생성 - 2
Root<Member> m = mainQuery.from(Member.class);
mainQuery.select(m).where(cb.ge(m.<Integer>get("age"), subQuery));
```

1. 서브 쿼리 생성 부분: 서브 쿼리는 `mainQuery.subquery(...)`로 생성
2. 메인 쿼리 생성 부분: `where(...subQuery)`에서 생성한 서브 쿼리를 사용

#### 상호 관련 서브 쿼리

Criteria에서 메인 쿼리와 서브 쿼리 간 서로 관련이 있을 때 서브 쿼리 작성하기.

서브 쿼리에서 메인 쿼리의 정보를 사용하려면 메인 쿼리에서 사용한 별칭을 얻어 사용한다. (메인 쿼리의 Root나 Join으로 생성된 별칭 사용)

`.where(cb.equal(subM.get("username"), m.get("username")))`

팀A에 소속된 회원 찾기(예제 위해 조인 아닌 서브 쿼리 사용)

```java
/*
  JPQL:
    select m from Member m
    where exists
      (select t from m.team t where t.name='팀A')
*/
CriteriaBuiler cb = em.getCriteriaBuiler();
CriteriaQuery<Member> mainQuery = cb.createQuery(Member.class);

//서브 쿼리에서 사용되는 메인 쿼리의 m
Root<Member> m = mainQuery.from(Member.class);

//서브 쿼리 생성
SubQuery<Team> subQuery = mainQuery.subquery(Team.class);
Root<Member> subM = subQuery.correlate(m); //메인 쿼리 별칭 가져옴

Join<Member, Team> t = subM.join("team");
subQuery.select(t).where(cb.equal(t.get("name"), "팀A"));

//메인 쿼리 생성
mainQuery.select(m).where(cb.exists(subQuery));

List<Member> resultList = em.createQuery(mainQuery).getResultList();
```

`correlate(...)`메소드로 메인 쿼리의 별칭을 서브 쿼리에서 사용.

### IN 식

IN 식은 Criteria 빌더에서 `in(...)`메소드 사용.

```java
/*
  JPQL
    select m from Member m
    where m.username in("회원1", "회원2")
*/
...
cq.select(m)
  .where(cb.in(m.get("username"))
  .value("회원1")
  .value("회원2"));
```

### CASE 식

CASE 식은 `selectCase()`, `when()`, `otherwise()` 메소드 사용.

```java
/*
  JPQL
    select m.usernmae,
      case when m.age>=60 then 600
      case when m.age<=15 then 500
           else 1000
      end
    from Member m
*/

Root<Member> m = cq.from(Member.class);

cq.multiselect(
  m.get("username"),
  cb.selectCase()
    .when(cb.ge(m.<Integer>get("age"), 60) 600)
    .when(cb.le(m.<Integer>get("age"), 15) 500)
    .otherwise(1000)
);
```

### 파라미터 정의

JPQL의 `:PARAM1`처럼 Criteria도 파라미터를 정의할 수 있다.

```java
/*
  JPQL
    select m from Member m
    where m.username = :usernameParam
*/

CriteriaBuiler cb = em.getCriteriaBuilder();
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);

//정의 - 1
cq.select(m)
  .where(cb.eqaul(m.get("username")m cb.parameter(String.class, "usernameParam")));

List<Member> resultList = em.createQuery(cq)
  .setParameter("username", "회원1") //바인딩 - 2
  .getResultList();
```

1. 파라미터 정의: `cb.parameter(타입, 파라미터이름)`
2. 파라미터에 사용할 값 바인딩: `setParameter("username", "회원1")`

### 네이티브 함수 호출

`cb.function(...)`메소드로 네이티브 SQL 함수 호출.

전체 회원 나이 합 구하는 예. "SUM" 대신 원하는 네이트 SQL 함수 입력.

```java
Root<Member> m = cq.from(Member.class);
Experession<Long> function = cb.function("SUM", Long.class, m.get("age"));
cq.select(function);
```

> JPQL과 마찬가지로 하이버네이트 구현체는 방언에 사용자정의 SQL 함수 등록해야 호출 가능

### 동적 쿼리

다양한 검색 조건에 따라 실행 시점에 쿼리를 생성하는 것을 동적 쿼리라 한다. 동적 쿼리는 문자 기반 보다 코드 기반인 Criteria로 작성하기 더 편리하다.  
-> 하지만 코드가 복잡해짐.

JPQL로 동적 쿼리를 작성 하려면 문자 더하기 연산으로 인한 버그 발생 가능성이 크다. (문자 사이 공백, where와 and 위치 구성 등)

### 함수 정리

Criteria는 JPQL의 빌더 역할을 하므로 JPQL 함수를 코드로 지원한다.

JPQL에서 사용하는 함수는 대부분 CriteriaBuilder에 정의되어 있다.

#### 조건 함수

| 함수명                       | JPQL                     |
| :--------------------------- | :----------------------- |
| and()                        | and                      |
| or()                         | or                       |
| not()                        | not                      |
| equal(), notEqual()          | =, \<>                   |
| lt(), lessThan()             | <                        |
| le(), lessThanOrEqualTo()    | <=                       |
| gt(), greaterThan()          | >                        |
| ge(), greaterThanOrEqualTo() | >=                       |
| between()                    | between                  |
| like(), notLike()            | like, not like           |
| isTrue(), isFalse()          | is true, is false        |
| in(), not(in())              | in, not(in())            |
| exists(), not(exists())      | exists, not exists       |
| isNull(), isNotNull()        | is null, is not null     |
| isEmpty(), isNotEmtpy()      | is empty, is not emtpy   |
| isMember(), isNotMember()    | member of, not member of |

#### 스칼라와 기타 함수

| 함수명 | JPQL | 함수명 | JPQL |
| :----- | :--- | :----- | :--- |

#### 집합 함수

| 함수명 | JPQL |
| :----- | :--- |

#### 분기 함수

| 함수명       | JPQL     |
| :----------- | :------- |
| nullif()     | nullif   |
| coalesce()   | coalesce |
| selectCase() | case     |

### Criteira 메타 모델 API

Criteira는 JPQL 코드 기반으로 작성한다. 그런데 `m.get("age")`처럼 직접 문자를 입력하는 부분이 생겨 완전한 코드 기반이라 할 수 없다.

이런 부분까지 코드로 작성하려면 메타 모델 API를 사용하면 된다. 메타 모델 API를 사용하려면 먼저 메타 모델 클래스를 만들어야 한다.

메타 모델 API 적용 예

- 메타 모델 API 적용 전

  ```JAVA
  cq.select(m)
    .where(cb.gt(m.<Integer>get("username"), 20))
    .orderBy(cb.desc(m.get("age")));
  ```

- 메타 모델 API 적용 후

  ```java
  cq.select(m)
    .where(cb.gt(m.get(Member_.age), 20))
    .orderBy(cb.desc(m.get(Member_.age)))
  ```

문자 기반 -> 정적 코드 기반으로 변경됨. 여기서 사용된 `Member_`클래스가 메타 모델 클래스이다.

`Member_` 메타 모델 클래스

```java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetaModel(Member.class)
public abstract class Member_ {

  public static volatile SingularAttribute<Member, Long> id;
  public static volatile SingularAttribute<Member, String> username;
  ...
  public static volatile ListAttribute<Member, Order> orders;
  public static volatile SingularAttribute<Member, Team> team;
}
```

이런 클래스를 표준(CANONICAL) 메타 모델 클래스라고 한다. 줄여서 메타 모델.

`Member_` 메타 모델 클래스는 코드 자동 생성기가 `Member` 엔티티 클래스를 기반으로 만들어준다.

코드 생성기는 모든 엔티티 클래스를 찾아 "엔티티명\_.java"모양의 메타 모델 클래스를 생성해준다.

엔티티 -> 코드 자동 생성기 -> 메타 모델 클래스

#### 코드 생성기 설정

코드 생성기는 보통 메이븐, 엔트, 그레이들 같은 빌드 도구를 사용해 실행한다.

메이븐 기준으로 두 설정을 추가 하고(dependency, plugin), `mvn compile` 명령어를 사용하면 target/generated-sources/annotations/ 하위에 메타 모델 클래스들이 생성된다.

Criteria를 자주 사용한다면 적용하는 것을 권장.

## QueryDSL

QueryDSL은 Criteria 보다 더 쉽고 간결하게, 쿼리와 비슷한 코드로 JPQL을 작성할 수 있다. QueryDSL은 오픈소스 프로젝트다. 지금은 JPA, MongoDB, 자바 컬렉션 등 다양하게 지원한다.

QueryDSL은 쿼리, 즉 데이터를 조회하는 데 기능이 특화되어 있다.

### QueryDSL 설정

#### 필요 라이브러리

예제 버전은 3.6.3 이다.

pom.xml에 QueryDSL 라이브러리 추가

- `querydsl-jpa`: QueryDSL JPA 라이브러리
- `querydsl-apt`: 쿼리 타입(Q) 생성할 때 필요한 라이브러리

#### 환경설정

QueryDSL을 사용하려면 엔티티를 기반으로 **쿼리 타입**이라는 **쿼리용 클래스를 생성**해야 한다. -> 쿼리 타입 생성용 플러그인 pom.xml에 추가.

플러그인 추가 후 콘솔에서 `mvn compile`을 입력하면 `outputDirectory`에 지정한 target/generated-sources위치에 `QMemeber.java`처럼 Q로 시작하는 쿼리 타입들이 생성된다.

#### 시작

QueryDSL 사용 간단한 예제

```java
public void queryDSL() {

  EntityManager em = emf.createEntityManager();

  JPAQuery query = new JPAQuery(em);
  QMember qMember = new QMember("m"); //생성되는 JPQL의 별칭이 m
  List<Member> members =
    query.from(qMember)
    .where(qMember.name.eq("회원1"))
    .orderBy(qMember.name.desc())
    .list(qMember);
}
```

QueryDSL를 사용하려면 우선 `com.mysema.query.jpa.impl.JPAQuery` 객체를 생성하면서 엔티티 매니저를 생성자에 넘겨준다. 다음으로 사용할 쿼리 타입(Q)을 생성하는데 생성자에는 별칭을 주면된다.

#### 기본 Q 생성

쿼리 타입(Q)는 사용하기 편리하도록 **기본 인스턴스를 보관**한다. 같은 엔티티를 조인 하거나 서브쿼리에 사용하면 같은 별칭이 사용되므로 이때는 별칭을 직접 지정해 사용해야 한다.

Member 쿼리 타입

```java
public class QMember extends EntityPathBase<Member> {

  public static final QMember member = new QMember("member1
  ");
  ...
}
```

쿼리 타입 사용

```java
QMember qMember = new QMember("m"); //직접 지정
QMember qMember = QMember.member; //기본 인스턴스 사용
```

쿼리 타입의 기본 인스턴스를 사용하면 `import static`을 사용해 더 간결하게 코드 작성 가능.

```java
import static jpabook.jpashop.domain.QMember.member; //기본 인스턴스

public void basic() {

  EntityManager em = emf.createEnittyManager();

  JPAQuery query = new JPAQuery(em);
  List<Member> members =
    query.from(member)
        .where(member.name.eq("회원1"))
        .orderBy(member.name.desc())
        .list(member);
}
```

### 검색 조건 쿼리

QueryDSL의 기본 쿼리 기능

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
    .where(item.name.eq("좋은상품").and(item.price.gt(20000)))
    .list(item); //조회할 프로젝션 지정
```

실행 JPQL

```SQL
select item
from Item item
where item.name = ?1 and item.price > ?2
```

QueryDSL의 where절에 and, or 사용 가능. 또는 여러 검색 조건을 사용하면(조건1, 조건, ...) and 연산이 된다.

쿼리 타입의 필드는 필요한 대부분의 메소드를 명시적으로 제공한다.  
(where절 사용 메소드 예: between, cotains, startWith ...)

### 결과 조회

쿼리 작성 후 결과 조회 메소드를 호출하면 실제 DB를 조회한다. 조회 메소드의 파라미터로 프로젝션 대상을 넘겨준다. 결과 조회 API는 `com.mysema.query.Projectable`에 정의되어 있다.

- `uniqueResult()`: 조회 결과가 한 건일 때 사용. 없으면 null을 반환. 결과가 하나 이상이면 NonUniqueResultException 예외 발생.
- `singleResult()`: 결과가 하나 이상이면 처음 데이터 반환.
- `list()`: 결과가 하나이상일 때 사용. 없으면 빈 컬렉션 반환.

### 페이징과 정렬

```java
QItem item = QItem.item;

query.from(item)
    .where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);
```

정렬은 `orderBy`로 쿼리 타입이 제공하는 `asc()`, `desc()`를 사용한다. 페이징은 `offset()`과 `limit()`을 조합해 사용하면 된다.

페이징은 `restric()` 메소드에 `com.mysema.query.QueryModifiers`를 파라미터로 사용해도 된다.

```java
QueryModifier queryModifiers = new QueryModifiers(20L, 10L); //limit, offset
List<Item> list =
    query.from(item)
    .restrict(queryModifiers)
    .list(item);
```

실제 페이징 처리할 때 검색된 전체 데이터 수를 알아야 하는데, 이 때는 `list()` 대신 `listResults()`를 사용한다.

`listResults()`를 사용하면 전체 데이터 조회를 위한 count 쿼리를 한 번 더 실행하고 `SearchResults`를 반환하는데 이 객체에서 전체 데이터 수를 조회한다.

### 그룹

`groupBy`를 사용. 그룹화된 결과 제한은 `having`을 사용.

```java
query.from(item)
  .groupBy(iteam.price)
  .having(item.price.gt(1000))
  .list(item);
```

### 조인

조인은 `innerJoin(join)`, `leftJoin`, `rightJoin`, `fullJoin`을 사용. JPQL의 on과 성능 최적화 위한 fetch 조인도 사용한다.

- 조인의 기본 문법
  - 첫 번째 파라미터: **조인 대상** 지정
  - 두 번째 파라미터: **별칭으로 사용할 쿼리 타입** 지정

기본적인 조인

```java
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
    .join(order.member, member)
    .leftJoin(order.orderItems, orderItem)
    .list(order);
```

조인에 ON 사용

```java
query.from(order)
    .leftJoin(order.orderItems, orderItem)
    .on(orderItem.count.gt(2))
    .list(order);
```

페치 조인

```java
query.from(order)
    .innerJoin(order.member, member).fetch()
    .leftJoin(orderI.orderItems, orderItem).fetch()
    .list(order);
```

세타 조인(from절에 여러 조인 사용)

```java
QOrder order = QOrder.order;
QMember member = QMember.member;

query.from(order, member)
    .where(order.member.eq(member))
    .list(order);
```

### 서브 쿼리

서브 쿼리는 `com.mysema.query.jpa.JPASubQuery`를 생성해 사용. 서브 쿼리 결과가 하나면 `unique()`, 여러 건이면 `list()`사용.

```java
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");

query.from(item)
    .where(item.price.eq(
        new JPQSubQuery().from(itemSub).unique(itemSub.price.max())
    ))
    .list(item);
```

여러 건의 서브 쿼리 사용

```java
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");

query.from(item)
    .where(item.in(
      new JPQSubQuery().from(itemSub)
        .where(item.name.eq(itemSub.name)
        .list(itemSub)
    ))
    .list(item);
```

### 프로젝션과 결과 반환

select절에 조회 대상을 지정하는 것을 프로젝션이라 한다.

#### 프로젝션 대상이 하나

프로젝션 대상이 하나면 해당 타입으로 반환한다.

```JAVA
QItem item = QItem.item;
List<String> result = query.from(item).list(item.name);

for (String name : result) {
  System.out.println("name = " + name);
}
```

#### 여러 컬럼 반환과 튜플

프로젝션 대상으로 여러 필드를 선택하면 QueryDSL은 기본으로 `com.mysema.query.Tuple`이라는 Map과 비슷한 내부 타입을 사용한다. 조회 결과는 `tuple.get()`에 조회한 쿼리타입을 지정.

튜플 사용 방법

```java
QItem item = QItem.item;

List<Tuple> result = query.from(item).list(item.name, item.price);
//List<Tuple> result = query.from(item).list(new QTuple(item.name, item.price));와 같다

for (Tuple tuple : result) {
  System.out.println("name = " + tuple.get(item.name));
  System.out.println("price = " + tuple.get(item.price));
}
```

#### 빈 생성

**쿼리 결과**를 엔티티가 아닌 **특정 객체로** 받으려면 빈 생성(Bean population)기능을 사용한다. QueryDSL은 객체를 생성하는 다양한 방법 제공.

- 프로퍼티 접근
- 필드 직접 접근
- 생성자 사용

원하는 방법 지정 위해 `com.mysema.query.type.Projections`를 사용한다. 다양한 방법으로 ItemDTO 값 채우기.

예제 ItemDTO

```java
public class ItemDTO {

  private String username;
  private int price;

  public ItemDTO() {}

  public ITemDTO(String usernmae, int price) {
    this.username = username;
    this.price = price;
  }

  //Getter, Setter
  public String getUsername() {...};
  public void setUsername() {...};
  public int getPrice() {...};
  public void setPrice() {...};
}
```

- 프로퍼티 접근

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
    Projections.bean(ItemDTO.class, item.name.as("username"), item.price));
```

`Projections.bean()` 메소드는 수정자를 사용해 값을 채운다. 쿼리 결과와 매핑할 프로퍼티 이름이 다르면 `as`를 사용해 별칭 지정. (쿼리 결과는 name, ItemDTO는 username 프로퍼티 가짐)

- 필드 직접 접근

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
    Projections.fields(ItemDTO.class, item.name.as("username"), item.price));
```

`Projections.bean()` 메소드는 필드에 직접 접근해 값을 채운다. 필드를 private로 설정해도 동작한다!

- 생성자 사용

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
    Projections.constructor(ItemDTO.class, item.name, item.price));
```

`Projections.constructor()` 메소드는 생성자를 사용한다.

#### DISTINCT

`query.distinct().from(item)...`처럼 DITSTINCT 사용.

### 수정, 삭제 배치 쿼리

QueryDSL도 수정, 삭제 배치 쿼리를 지원한다. JPQL 배치 쿼리와 같이 **영속성 컨텍스트를 무시하고 데이터베이스를 직접 쿼리**한다는 점을 유의하자.

수정 배치 쿼리

```java
QItem item = QItem.item;
JPAUpdateClause updateClause = new JPQUpdateClause(em, item);
long count = updateClause.where(item.name.eq("시골개발자의 JPA 책"))
  .set(item.price, item.price.add(100))
  .execute();
```

수정 배치 쿼리는 `com.mysema.query.jpa.impl.JPQUpdateClause`를 사용. (상품 가격 100원 증가)

```java
QItem item = QItem.item;
JPADeleteClause deleteClause = new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("시골개발자의 JPA 책"))
  .execute();
```

삭제 배치 쿼리는 `com.mysema.query.jpa.impl.JPQDeleteClause`를 사용. (이름 같은 상품 삭제)

### 동적 쿼리

`com.mysema.query.BooleanBuilder`로 특정 조건에 따른 동적 쿼리를 편리하게 생성할 수 있다.

동적 쿼리 생성

```java
SearchParam param = new SearchParam();
param.setName("시골개발자");
param.setPrice(10000);

QItem item = QItem.item;

BooleanBuilder builder = new BooleanBuilder();
if (StringUtils.hasNext(param.getName())) {
  builder.and(item.name.contains(param.getName()));
}
if (param.getPrice() != null) {
  builder.and(item.price.gt(param.getPrice()));
}
List<Item> result = qeury.from(item)
    .where(builder)
    .list(item);
```

예제는 상품 이름과 가격 유무에 따라 동적 쿼리를 생성한다.

### 메소드 위임

메소드 위임(Delegate methods) 기능을 사용하면 **쿼리 타입에 검색 조건을 직접 정의**할 수 있다.

메소드 위임 기능을 사용하려면 우선 정적 메소드를 만들고 `@com.mysema.query.annotations.QueryDelegate` 어노테이션에 속성으로 이 기능을 적용할 엔티티를 지정한다. 정적 메소드의 첫 번째 파라미터는 대상 엔티티의 쿼리 타입(Q)을 지정하고, 나머지는 필요한 파라미터를 정의한다.

검색 조건 정의

```java
public class ItemExpression {

  @QueryDelegate(item.class)
  public static BooleanExpression isExpensive(QItem item, Integer price) {
    return item.price.gt(price);
  }
}
```

쿼리 타입에 생성된 결과

```java
public class QItem extends EntityPathBase<Item> {
  ...
  public com.mysema.query.types.expr.BooleanExpression isExpensive(Integer price) {
    return ItemExpression.isExpensive(this, price);
  }
}
```

메소드 위임 기능 사용

`query.from(item).where(item.isExpensive(30000)).list(item);`

필요하면 자바 기본 내장 타입(String, Date ...)에도 메소드 위임 기능 사용 가능.

```java
@QueryDelegate(String.class)
public static BooleanExpression isHelloStart(StringPath stringPath) {
  return stringPath.startsWith("Hello");
}
```

### QueryDSL 정리

JPQL을 코드로 작성하거나 쉽게 동적 쿼리를 생성하고 싶다면 복잡한 Criteria 보다는 직관적이고 단순한 QueryDSL을 사용하자!

## 네이티브 SQL

JPQL이 지원하지 않는 특정 데이터베이스에 종속적인 기능을 사용하려면 JPA가 지원하는 다양한 방법을 이용하면 된다. (JPA 구현체들이 표준보다 더 다양한 방법 지원)

- **특정 데이터베이스만 사용하는 함수**
  - JPQL에서 네이티브 SQL 함수 호출.(JPA 2.1)
  - 하이버네이트는 DB 방언에 각 데이터베이스에 종속적인 함수들을 정의해 둠. 직접 호출할 함수 정의 가능.
- **특정 데이터베이스만 지원하는 SQL 쿼리 힌트**
  - 하이버네이트 포함 몇몇 JPA 구현체들이 지원.
- **인라인 뷰(From절 서브쿼리), UNION, INTERSECT**
  - 하이버네이트X. 일부 JPA 구현체들이 지원.
- **스토어드 프로시저**
  - JPQL에서 스토어드 프로시저 호출.(JPA 2.1)
- **특정 데이터베이스만 지원하는 문법**
  - 오라클 CONNECT BY처럼 특정 DB에 너무 종속적인 SQL 문법은 지원 않는다. 이때 네이티브 SQL 사용.

다양한 이유로 JPQL을 사용할 수 없을 때 JPA는 SQL을 직접 사용할 수 있는 기능을 제공하는데 이것을 네이티브 SQL이라 한다. JPQL을 사용하면 JPA가 SQL을 생성해주는데, 개발자가 JPA 대신 SQL을 직접 정의하는 것이다.

Q: JPA가 지원하는 네이티브 SQL과 JDBC API를 직접 사용하는 것의 차이는?  
 => **네이티브 SQL은** 엔티티 조회 가능, JPA가 지원하는 **영속성 컨텍스트의 기능을 그대로 사용**. JDBC API를 직접 사용하면 단순한 데이터 나열을 조회하는 것.

### 네이티브 SQL 사용

네이티브 쿼리 API 3가지.

```JAVA
//결과 타입 정의
public Query createNativeQuery(String sqlString, Class resultClass;)

//결과 타입을 정의할 수 없을 때
public Query createNativeQuery(String sqlString);

public Query createNativeQuery(String sqlString, String resultSetMapping);
```

#### 엔티티 조회

`em.createNativeQuery(SQL, 결과클래스)`를 사용해 엔티티를 조회한다. JPQL과 비슷하지만 실제 데이터베이스 SQL을 사용한다는 것과 위치기반 파라미터만 지원한다는 차이가 있다.

가장 중요한 점은 네이티브 SQL로 SQL만 직접 사용할 뿐, 나머지는 JPQL을 사용할 때와 같다. **조회한 엔티티도 영속성 컨텍스트에서 관리된다.**

> 하이버네이트는 네이티브 SQL에 이름 기반 파라미터 사용 가능.

> `em.createNativeQuery()`를 호출하며 타입 정보를 주었어도 TypeQuery가 아닌 Query를 리턴. 그냥 규약이 그렇다.

#### 값 조회

단순히 값 조회하기.

```java
//SQL 정의
String sql =
    "SELECT ID, AGE, NAME, TEAM_ID " +
    "FROM MEMBERSS WHERE AGE > ?";

Query nativeQuery = em.createNativeQuery(sql)
    .setParameter(1, 10);

List<Object[]> resultList = nativeQuery.getResultList();
for (Object[] row : resultList) {
  System.out.println("id = " + row[0]);
  System.out.println("age = " + row[1]);
  System.out.println("name = " + row[2]);
  System.out.println("team_id = " + row[3]);
}
```

엔티티로 조회하지 않고 단순히 값으로 조회했다. 여러 값으로 조회 하려면 `em.createNativeQuery(SQL)`의 두 번째 파라미터를 사용 않으면 된다. JPA는 조회한 값들을 Object[]에 담아 반환한다. 여기서는 스칼라 값들을 조회했을 뿐이므로 영속성 컨텍스트가 결과를 관리하지 않는다. JDBC로 데이터 조회한 것과 비슷하다.

#### 결과 매핑 사용

엔티티 + 스칼라 값 조회처럼 매핑이 복잡해지면 `@SqlResultSetMapping`을 정의해서 결과 매핑을 사용해야 한다. 두 번째 파라미터에 결과 매핑 정보의 이름 사용.

회원 엔티티 + 회원이 주문한 상품 수 조회

```java
//SQL 정의
String sql =
    "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
    "FROM MEMBER M " +
    "LEFT JOIN " +
    "   (SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
    "   FROM ORDERS O, MEMBER IM " +
    "   WHERE O.MEMBER_ID = IM.ID) I " +
    "ON M.ID = I.ID";

Query nativeQuery = em.createNativeQuery(sql, "memberWithOrderCount");
List<Object[]> resultList = nativeQuery.getResultList();
for (object[] row : resultList) {
  Member member = (Member) row[0];
  BigInteger orderCount = (BigInteger)row[1];

  System.out.println("member = " + member);
  System.out.println("orderCount = " + orderCount);
}
```

결과 매핑 정의

```java
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount",
    entities = {@EntityResult(EntityClass = Member.class)},
    columns = {@ColumnResult(name = "ORDER_COUNT")}
)
public class Member {...}
```

`memberWithOrderCount`의 결과 매핑을 보면 회원엔티티와 `ORDER_COUNT` 컬럼을 매핑함. 조회 쿼리 결과에서 ID, AGE, NAME, TEAM_ID는 Member 엔티티와 매핑하고 ORDER_COUNT는 단순 값으로 매핑한다.

`@EntityResult`에서 `fields={@FieldResult(name="", column="")}`으로 컬럼명과 필드명을 직접 매핑한다. 이 설정은 엔티티의 필드에 정의한 `@Column`보다 앞선다. 두 엔티티를 조회할 때 컬럼명이 중복될 때도 `@FieldResult`를 사용한다.

#### 결과 매핑 어노테이션

결과 매핑에 사용하는 어노테이션.

- `@SqlResultSetMapping`
- `@EntityResult`
- `@FieldResult`
- `@ColumnResult`

### Named 네이티브 SQL

네이티브 SQL도 Named 네이티브 SQL을 사용해 정적 SQL을 작성할 수 있다.

엔티티 조회

```java
@Entity
@NamedNativeQuery(
    name = "Member.memberSQL",
    query = "SELECT ID, AGE, NAME, TEAM_ID " +
        "FROM MEMBER WHERE AGE > ?",
    resultClass = Member.class
)
public class member {...}
```

`@NamedNativeQuery`로 Named 네이티브 SQL을 등록한다.

Named 네이티브 SQL 사용

```java
TypedQuery<Member> nativeQuery =
    em.createNativeQuery("Member.memberSQL", Member.class)
        .setParameter(1, 20);
```

JPQL과 같은 `createNativeQuery` 메소드를 사용한다. 따라서 `TypedQuery`를 사용할 수 있다.

Named 네이티브 SQL에서 결과 매핑 사용

```java
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount",
    entities = {@EntityResult(EntityClass = Member.class)},
    columns = {@ColumnResult(name = "ORDER_COUNT")}
)
@NamedNativeQuery(
    name = "Member.memberWithOrderCount",
    query = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
      "FROM MEMBER M " +
      "LEFT JOIN " +
      "   (SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
      "   FROM ORDERS O, MEMBER IM " +
      "   WHERE O.MEMBER_ID = IM.ID) I " +
      "ON M.ID = I.ID",
    resultSetMapping = "memberWithOrderCount"
)
public class Member {...}
```

Named 네이티브 쿼리에서 `resultSetMapping = "memberWithOrderCount"`으로 조회 결과를 매핑할 대상 지정.

정의한 Named 네이티브 쿼리 사용 코드

```java
List<Obejct[]> resultList =
    em.createNamedQuery("Member.memberWithOrderCount")
      .getResultList();
```

### 네이티브 SQL XML에 정의

Named 네이티브 쿼리 XML에 정의

ormMember.xml

```xml
<entity-mappings ...>

    <named-native-query name ="Member.memberWithOrderCountXml" result-set-mapping="memberWithOrderCountResultMap">
      <query><CDATA[
        SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT
        FROM MEMBER M
        LEFT JOIN
          (SELECT IM.ID, COUNT(*) AS ORDER_COUNT
          FROM ORDERS O, MEMBER IM
          WHERE O.MEMBER_ID = IM.ID) I
        ON M.ID = I.ID
      ]></query>
    </named-native-query>

    <sql-result-set-mapping name="memberWithOrderCountResultMap">
      <entity-result entity-class="jpabook.domain.Member" />
      <column-result name="ORDER_COUNT" />
    </sql-result-set-mapping>

</entity-mappings>
```

XML에 정의할 때 순서를 지켜서 `<named-native-query>`, `<sql-result-set-mapping>` 순으로 작성. (사용 코드는 동일)

> 네이티브 SQL은 복잡한 SQL 작성, SQL 최적화해 DB 성능 향상 위해 사용함. 이런 쿼리들은 복잡하고 라인수가 많다. -> 어노테이션보다 XML을 사용하면 편리하다.

### 네이티브 SQL 정리

네이티브 SQL도 JPQL처럼 `Query, TypeQuery`를 반환한다. 따라서 JPQL API를 그대로 사용 가능. (페이징 처리 API 등)

네이티브 SQL은 JPQL이 자동 생성 해주는 SQL을 수동으로 정의하는 것이다. 그래서 JPA의 기능을 대부분 사용할 수 있다.

하지만 특정 DB에 종속적인 쿼리가 증가해 이식성이 떨어지기 때문에 필요할 때에만 사용해야 한다.

1. 가능하면 표준 JPQL 사용
2. 기능 부족 시 하이버네이트같은 JPQ 구현체 사용
3. 안되면 마지막 방법으로 네이티브 SQL 사용
4. 네이티브 SQL도 부족하면 MyBatis, JdbcTemplate같은 SQL 매퍼와 JPA를 함께 사용

### 스토어드 프로시저

#### 스토어드 프로시저 사용

단순히 입력 값을 두 배로 증가시키는 스토어드 프로시저 `proc_multiply`가 있다. (첫 번째 파라미터로 값 입력 받고, 두 번째 파라미터로 결과 반환)

`proc_multiply` MySQL 프로시저

```SQL
DELIMITER //

CREATE PROCEDURE proc_multiply (INOUT inParam INT, INOUT outParam INT)
BEGIN
    SET outParma = inParam * 2;
ENT //
```

JPA로 스토어드 프로시저 호출 - 순서 기반 파라미터 호출 코드

```JAVA
storedProcedureQuery spq =
  em.createStoredProcedureQuery("proc_multiply");
spq.registerStoredProcedureParameter(1, Integer.class,
    ParameterMode.IN);
spq.registerStoredProcedureParameter(2, Integer,class,
    ParameterMode.OUT);

spq.setParameter(1, 100);
spq.execute();

Integer out = (Integer)spq.getOutputParameterValue(2);
System.out.println("out = " + out); //결과 = 200
```

스토어드 프로시저 사용 위해 `em.createStoredProcedureQuery()`메소드에 사용할 스토어드 프로시저 이름 입력. `registerStoredProcedureParameter()`로 프로시저에서 사용할 파라미터를 순서, 타입, 파라미터 모드 순 정의.

- ParameterMode
  ```java
  public enum ParameterMode {
    IN,        //INPUT 파라미터
    INOUT,     //INPUT, OUTPUT 파라미터
    OUT,       //OUTPUT 파라미터
    REF_CURSOR //CURSOR 파라미터
  }
  ```

JPA로 스토어드 프로시저 호출 - 이름 기반 파라미터 호출 코드

```JAVA
storedProcedureQuery spq =
  em.createStoredProcedureQuery("proc_multiply");
spq.registerStoredProcedureParameter("inParam", Integer.class, ParameterMode.IN);
spq.registerStoredProcedureParameter("outParam", Integer,class, ParameterMode.OUT);

spq.setParameter("inParam", 100);
spq.execute();

Integer out = (Integer)spq.getOutputParameterValue("outParam");
System.out.println("out = " + out); //결과 = 200
```

#### Named 스토어드 프로시저 사용

스토어드 프로시저 쿼리에 이름을 부여해서 사용하는 것을 Named 스토어드 프로시저라 한다.

Named 스토어드 프로시저 어노테이션에 정의하기

```java
@NamedStoredProcedureQuery(
    name = "multiply",
    procedureName = "proc_multiply",
    parameters = {
      @StoredProcedureParameter(name = "inParam", mode =
          ParameterMode.IN, type = Integer.class),
      @StoredProcedureParameter(name = "outParam", mode =
          ParameterMode.OUT, type = Integer.class),
    }
)
@Entity
public class Member {...}
```

`@NamedStoredProcedureQuery`로 정의하고 name 속성으로 이름을 부여한다. `procedureName` 속성에 실제 호출할 프로시저 이름을 적고 `@StoredProcedureParameter`로 파라미터 정보를 정의한다.

Named 스토어드 프로시저 XML에 정의하기

```xml
<?xml version="1.0" encoding="UTF-8">
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">

    <named-stored-procedure-query name="multiply" procedure-name="proc_multiply">
      <parameter name="inParam" mode="IN" class="java.lang.Integer" />
      <parameter name="outParam" mode="OUT" class="java.lang.Integer" />
    </named-stored-procedure-query>

</entity-mappings>
```

xml로 정의한 Named 스토어드 프로시저는 `em.createNamedStoredProcedureQuery()` 메소드에 등록한 Named 스토어드 프로시저 이름을 파라미터로 사용해 찾아올 수 있다.

Named 스토어드 프로시저 사용

```java
StoredProcedureQuery spq =
    em.createNamedStoredProcedureQuery("multiply");

spq.setParameter("inParam", 100);
spq.execute();

Integer out = (Integer) spq.getOutputParameterValue("outParam");
System.out.println("out = " + out);
```

## 객체지향 쿼리 심화

객체지향 쿼리와 관련된 고급 주제. 벌크 연산, JPQL과 영속성 컨텍스트, JPQL과 플러시 모드.

### 벌크 연산

여러 건을 한 번에 수정, 삭제하는 벌크 연산 사용하기. 영속성 컨텍스트의 변경 감지 기능, 병합 사용, 삭제 메소드 사용 등의 과정을 수백개 이상의 엔티티가 처리 되는 시간을 줄이기 위해 사용한다.

벌크연산 사용 - 재고가 10개 미만인 모든 상품 가격 10% 상승

```java
String qlString =
    "update Product p " +
    "set p.price = p.price * 1.1 " +
    "where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
                    .setParameter("stockAmount", 10)
                    .executeUpdate();
```

벌크 연산은 `executeUpdate()`를 사용한다. 이 메소드는 벌크 연산으로 영향을 받은 **엔티티 건수**를 반환한다.

벌크연산 사용 - 가격이 100원 미만인 상품 삭제

```java
String qlString =
    "delete Product p " +
    "where p.price < :price";

int resultCount = em.createQuery(qlString)
                    .setParameter("price", 100)
                    .executeUpdate();
```

> 하이버네이트는 INSERT 벌크 연산도 지원한다.

#### 벌크 연산의 주의점

벌크 연산을 사용할 때 **벌크 연산이 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리**한다는 점에 주의해야 한다. 벌크 연산 시 발생하는 문제를 예제 통해 알아보기.

벌크 연산 시 주의점 예제 - DB에 가격 1000원인 상품 A 존재

```java
//상품 조희(상품A의 가격은 1000원이다.)
Product productA =
    em.createQuery("select p from Product p where p.name = :name", Product.class)
    .setParameter("name", "productA")
    .getSingleResult();

//출력 결과: 1000
System.out.println("productA 수정 전 = " + productA.getPrice());

//벌크 연산 수행으로 모든 상품 가격 10% 상승
em.createQuery("update Product p set p.price = p.price * 1.1").executeUpdate();

//출력 결과: 1000
System.out.println("productA 수정 후 = " + productA.getPrice());
```

-> 벌크 연산 수행 결과 productA의 상품 가격이 기대한 1100원이 아니고 1000원으로 출력된다!  
-> 벌크 연산은 영속성 컨텍스트를 통하지 않고 데이터베이스에 직접 쿼리한다.(처음 상품 조회 시 100원인 상품A가 영.컨에 관리됨, 벌크연산은 DB의 데이터를 대상으로 수행)  
=> 영속성 컨텍스트의 상품A와 데이터베이스의 상품A의 가격이 다를 수 있다.

#### 벌크 연산 문제점 해결 방법

- **`em.refresh()`사용**  
  벌크 연산 수행 직후 정확한 상품A 엔티티를 사용하려면 `em.refresh()`를 사용해 DB에서 상품A를 다시 조회하면 된다.  
  `em.refresh(productA)`
- **벌크 연산 먼저 수행**  
  가장 실용적인 해결책은 벌크 연산을 먼저 수행하는 것이다. 벌크 연산을 먼저 실행 후 상품A를 조회하면 이미 변경된 상품을 조회하게 된다. JPA + JDBC를 함께 사용할 때 유용한 방법.
- **벌크 연산 수행 후 영속성 컨텍스트 초기화**  
  벌크 연산 수행 직후 바로 영속성 컨텍스트를 초기화해 영속성 컨텍스트에 남아 있는 엔티티를 제거 하는 것도 좋은 방법이다. 영속성 컨텍스트를 초기화하면 이후 엔티티 조회 시 벌크 연산이 적용된 DB에서 엔티티를 조회한다.

-> 벌크 연산은 영속성 컨텍스트와 2차 캐시를 무시하고 DB에 직접 실행한다.  
-> 영속성 컨텍스트와 DB간 데이터 차이 발생 가능하므로 주의 해서 사용  
-> 가능하면 벌크 연산을 가장 먼저 수행 + 상황따라 영속성 컨텍스트 초기화

### 영속성 컨텍스트와 JPQL

#### 쿼리 후 영속 상태인 것과 아닌 것

JPQL의 조회 대상: 엔티티, 임베디드 타입, 값 타입

JPQL로 엔티티 조회 시 영속성 컨텍스트에서 관리. 엔티티가 아니면 영컨에서 관리X.

예를 들어 임베디드 타입은 값을 변경해도 영속성 컨텍스트의 변경 감지에 의한 수정이 발생하지 않는다.

-> 조회한 엔티티만 영속성 컨텍스트가 관리

#### JPQL로 조회한 엔티티와 영속성 컨텍스트

🙋‍♀️영속성 컨텍스트에 회원1이 이미 있는데, JPQL로 회원1을 다시 조회한다면?

```java
`em.find(Memeber.class, "member1")` //회원1 조회

//엔티티 쿼리 조회 결과가 회원1, 회원2
List<Member> resultList = em.createQuery("select m from Member m", Member.class).getResultList();
```

✔️JPQL로 DB에서 조회한 엔티티가 영속성 컨텍스트에 이미 있으면 JPQL로 **데이터베이스를 조회한 결과를 버리고, 대신에 영속성 컨텍스트에 있던 데이터를 반환**한다. 이때 **식별자 값을 사용해 비교**한다.

- 회원1이 영컨에 이미 존재하고, 회원1과 회원2를 JPQL로 조회할 때.

1. JPQL을 사용해 조회를 요청
2. JPQL은 SQL로 변환되어 데이터베이스를 조회
3. 조회한 결과와 영속성 컨텍스트를 비교 _(식별자 값으로 비교)_
4. 식별자 값 기준으로 이미 영속성 컨텍스트에 엔티티가 있으면 버리고, 기존에 있던 회원1이 반환 대상이 된다 _(기존 영속성 컨텍스트의 엔티티 반환)_
5. 식별자 값 기준으로 회원2는 영속성 컨텍스트에 없으므로 영속성 컨텍스트에 추가
6. 쿼리 결과인 회원1, 회원2를 반환. 여기서 회원1은 쿼리 결과가 아닌 영속성 컨텍스트에 있던 엔티티다

🙋‍♀️왜 DB 조회 값을 버리고 기존 엔티티를 반환하는 것일까? JPQL로 조회한 새로운 엔티티를 영속성 컨텍스트에 하나 더 추가 or 기존 엔티티를 새로 검색한 엔티티로 대체한다면?

✔️영속성 컨텍스트는 기본 키 값을 기준으로 엔티티를 관리한다. 같은 기본 키 값인 엔티티는 등록할 수 없다. (1번 X)

2번은 영속성 컨텍스트에 수정중인 데이터가 사라질 수 있으므로 위험하다. (2번 X)

**영속성 컨텍스트는 엔티티의 동일성을 보장한다.** 따라서 영속성 컨텍스트는 3번으로 동작한다. (3번 O)

1. 새로운 엔티티를 영속성 컨텍스트에 하나 더 추가 -> X
2. 기존 엔티티를 새로 검색한 엔티티로 대체 -> X
3. 기존 엔티티는 그대로 두고 새로 검색한 엔티티 버리기 -> O

-> `em.find()`나 JPQL로 조회해도 둘 다 영속성 컨텍스트가 같으면 동일한 엔티티를 반환한다.

#### find() vs JPQL

`em.find()`는 엔티티를 먼저 영속성 컨텍스트에서 찾고 없으면 데이터베이스에서 찾는다. 엔티티가 영속성 컨텍스트에 있다면 메모리에서 바로 찾으므로 성능상 이점이 있다. (1차 캐시)

JPQL은 `em.find()`를 2번 사용한 로직과 마찬가지로 주소 값이 같은 인스턴스를 반환하고 결과도 같다. 하지만 내부 동작 방식을 조금 다르다.

**JPQL은 항상 데이터베이스에 SQL을 실행해 결과를 조회**한다.

```JAVA
//첫 번째 호출: 데이터베이스에서 조회
Member member1 =
    em.createQuery("select m from Member m where m.id = :id", Member.class)
      .setParameter("id", 1L)
      .getSingltResult();

//두 번째 호출: 데이터베이스에서 조회
Member member2 =
    em.createQuery("select m from Member m where m.id = :id", Member.class)
      .setParameter("id", 1L)
      .getSingltResult();

//member1 == member2는 주소값이 같은 인스턴스
```

첫 번째 JPQL을 호출하면 데이터베이스에서 회원 엔티티를 조회하고 영속성 컨텍스트에 등록한다. 두 번째 JPQL을 호출하면 데이터베이스에서 같은 회원 엔티티를 조회한다. 이때 영속성 컨텍스트에 이미 조회한 같은 엔티티가 있으므로, 새로 검색한 엔티티는 버리고 영속성 컨텍스트에 있는 기존 엔티티를 반환한다.

#### JPQL의 특징 정리

1. JPQL은 항상 데이터베이스를 조회한다.
2. JPQL로 조회한 엔티티는 영속 상태다.
3. 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티를 반환한다.

### JPQL과 플러시 모드

#### 플러시 복습

**플러시:** 영속성 컨텍스트의 변경 내역을 데이터베이스에 동기화 하는 것이다. JPA는 플러시가 일어날 때 영속성 컨텍스트에 등록, 수정, 삭제한 엔티티를 찾아 SQL을 만들어 데이터베이스에 반영한다.

플러시 호출: `em.flush()`직접 호출 \/ 보통 플러시 모드에 따라 커밋 직전이나 쿼리 실행 직전에 자동으로 플러시 호출됨.

`setflushMode(FlushModeType.AUTO)` //커밋 또는 쿼리 실행 시 플러시(기본 값)  
`setflushMode(FlushModeType.COMMIT)` //커밋 시 플러시

플러시모드는 기본값을 사용하고, 커밋 시 플러시는 성능 최적화를 위해 꼭 필요할 때만 사용해야 한다.

#### 쿼리와 플러시 모드

JPQL은 영속성 컨텍스트의 데이터를 고려 하지 않고, 데이터베이스에서 데이터를 조회한다. 따라서 JPQL을 실행하기 전에 영속성 컨텍스트의 내용을 데이터베이스에 반영해야 한다.

만약 플러시 모드가 COMMIT일 때 영속성 컨텍스트의 내용이 아직 플러시 되지 않아 데이터베이스에 반영되지 않으면, 쿼리시에는 플러시 하지 않으므로 방금 수정한 데이터를 조회할 수 없다. 이때는 직접 `em.flush()`를 호출하거나 **Query 객체에 플러시 모드를 설정**해주면 된다.

Query 객체에 플러시 모드 설정하기

```java
em.setFlushMode(FlushModeType.COMMIT)

//가격을 1000 -> 2000으로 변경
product.setPrice(2000);

//1. em.flush

//가격이 2000인 상품 조회
Product product2 =
    em.createQuery("". Product.class)
        .setFlushMode(FlushModeType.AUTO) //2.setFlushMode() 설정
        .getSingleResult();
```

쿼리 실행 전에 플러시를 호출하려면 1을 사용해 수동으로 플러시 하거나, 2로 해당 쿼리에서만 사용할 플러시 모드를 설정하면 된다.이렇게 쿼리에 설정하는 플러시모드는 엔티티 매니저에 설정하는 플러시 모드보다 우선권을 가진다.

#### 플러시 모드와 최적화

커밋 플러시 모드는 트랜잭션을 커밋할 때만 플러시하고 쿼리를 실행할 때는 플러시하지 않는다. 따라서 JPA 쿼리를 사용할 때 영속성 컨텍스트에는 있지만 아직 DB에 반영하지 않은 데이터를 조회할 수 없다. -> 데이터 무결성에 피해 가능성O

그럼에도 플러시가 너무 자주 일어나는 상황에 커밋 플러시 모드를 사용하면 **쿼리시 발생하는 플러시 횟수를 줄여 성능을 최적화**할 수 있다.

JPA가 아닌 JDBC를 직접 사용해 SQL을 실행할 때도 플러시 모드를 고민해야 한다. JDBC로 쿼리를 직접 실행하면 JPA는 JDBC가 실행한 쿼리를 인식할 방법이 없다. 따라서 별도의 JDBC 호출은 플러시 모드를 AUTO 설정해도 플러시가 일어나지 않는다. 이때는 JDBC로 쿼리 실행 직전 `em.flush()`를 호출해 영속성 컨텍스트의 내용을 DB에 동기화 하는 것이 안전하다.

## 정리

- JPQL은 SQL을 추상화해 특정 데이터베이스 기술에 의존하지 않는다.
- Criteria나 QueryDSL은 JPQL을 만들어주는 빌더 역할을 할 뿐이므로 JPQL을 잘 알아야 한다.
- Criteria나 QueryDSL을 사용하면 동적 쿼리를 편리하게 작성할 수 있다.
- Criteria는 JPA가 공식 지원하는 기능 이지만 사용하기 불편하다. 반면 QueryDSL은 JPA가 공식 지원하는 기능은 아니지만 직관적이고 편리하다.
- JPA도 네이티브 SQL을 제공하므로 직접 SQL을 사용할 수 있다. 최대한 JPQL을 사용하고, 방법이 없을 때 사용하면 된다.
- JPQL은 대량의 데이터를 수정, 삭제하는 벌크 연산을 지원한다.
