# 2. JPA 시작

## 라이브러리

#### 하이버네이트 핵심 라이브러리

- hibernate-core: 하이버네이트 라이브러리
- hibernate-enitymanager: 하이버네이트가 jpa 구현체로 동작하도록 jpa 표준 구현한 라이브러리
- hibernate-jpa-x.x-api: 해당 버전의 JPA 표준 API를 모아둔 라이브러리

> 하이버네이트: JPA 구현체, 메이븐: 라이브러리 관리 도구

## 객체 매핑

JPA는 매핑 어노테이션을 분석해 어떤 객체가 어떤 테이블과 관계가 있는지 알아낸다.

#### JPA 매핑 어노테이션

- @Entity: 클래스를 테이블과 매핑. -> 엔티티 클래스
- @Table: 엔티티 클래스에 매핑할 테이블 정보. 생략시 클래스 이름을 테이블 이름으로 매핑.
- @Id: 엔티티 클래스의 필드를 테이블의 기본 키에 매핑. -> 식별자 필드
- @Column: 필드를 컬럼에 매핑.
- 매핑 정보가 없는 필드: 매핑 어노테이션 생략시 필드명을 컬럼명으로 매핑.

## JPA 설정 정보 관리

JPA는 `persistence.xml`이라는 환경설정파일을 사용해 필요한 설정 정보를 관리한다.

#### <persistence ~>

xml 네임스페이스, 사용할 버전을 명시한다.

#### <persistence-unit ~>

JPA 설정은 일반적으로 연결할 **데이터베이스당 하나의 영속성 유닛(persistence-unit)을 등록**한다. 영속성 유닛에는 고유한 이름을 부여한다. 영속성 유닛에서 속성(properties)을 설정한다.

#### <protperties>

- JPA 표준 속성(필수 속성으로 특정 구현체에 종속되지 않음)
  - javax.persistence.jdbc.driver: JDBC 드라이버
  - javax.persistence.jdbc.user: 데이터베이스 접속 아이디
  - javax.persistence.jdbc.password: 데이터베이스 접속 비밀번호
  - javax.persistence.jdbc.url: 데이터베이스 url
- 하이버네이트 속성
  - hibernate.dialect: 데이터베이스 방언(Dialect) 설정

#### 데이터베이스 방언 Dialect

SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서는 방언(Dialect)이라고 한다.  
JPA는 특정 데이터베이스에 종속적이지 않은 기술이다. 방언을 많이 사용하면 DB의 교체가 어려워지는데 이런 문제 해결을 위해 JPA의 구현체들은 다양한 데이터베이스 방언 클래스를 제공한다.

> **하이버네이트 DB 방언**  
> H2: org.hibernate.dialect.H2Dialect  
> 오라클 10g: org.hibernate.dialect.Oracle10gDialect  
> MySQL: org.hibernate.dialect.MySQL5InnoDBDialect

## JPA 사용한 애플리케이션 개발

JPA를 사용한 애플리케이션 개발은 크게 세 부분으로 나뉜다.  
1.엔티티 매니저 설정, 2.트랜잭션 관리, 3.비즈니스 로직

#### 엔티티 매니저 설정

엔티티 매니저의 생성 과정

1. **설정 정보 조회(persistence.xml)**
2. **엔티티매니저팩토리 생성**

- Persistence 클래스로 엔티티 매니저 팩토리를 생성해 JPA를 사용할 수 있게 준비.(설정정보로 기반 객체 만들기, DB 커넥션 풀 생성 등)
- 엔티티팩토리매니저 생성 비용은 아주 큼 -> 엔티티매니저팩토리는 애플케이션 전체에서 한 번만 생성하고 공유해 사용.

3. **엔티티매니저 생성**

- JPA의 기능 대부분 제공(엔티티를 DB에 crud)
- 내부에 데이터소스(DB 커넥션)을 유지하면서 DB와 통신
- DB 커넥션과 밀접한 관계 -> 스레드간 공유, 재사용 안됨

4. **종료**

- 사용이 끝난 엔티티 매니저 종료
- 애플리케이션 종료할 때 엔티티매니저팩토리 종료

```JAVA
EntityManagerFactory emf = Persistence.createEntityManagerFactory("영속성유닛이름");

EntityManmanager em = emf.createEntityManager();

em.close();
emf.close();
```

#### 트랜잭션 관리

JPA는 트랜잭션 안에서 데이터를 변경해야 한다. 트랜잭션을 시작하려면 엔티티 매니저에서 트랜잭션 API를 받아온다.

```JAVA
EntityTransaction tx = em.getTransaction();
try {
  tx.begin();
  login(em);
  tx.commit();
} catch (Exception e) {
  tx.rollback();
}
```

#### 비즈니스 로직

엔티티 매니저를 통해 데이터베이스에 등록, 수정, 삭제, 조회 하기.

- 등록 `em.persist(Object)`: JPA는 엔티티의 매핑정보를 분석해 SQL 만들어 전달.
- 수정 `o.setAge(~)`: 엔티티의 값을 변경하면 JPA가 UPDATE SQL을 생성해 DB에 전달.
- 삭제 `em.remove(Object)`: JPA가 DELETE SQL을 생성해 실행.
- 한 건 조회 `em.find(Obejct.class, 식별자 필드)`: SELECT SQL을 생성해 DB에 전달하고 조회한 결과 값으로 엔티티를 생성해서 반환.

#### JPQL (Java Persistence Query Language)

JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다.  
필요한 데이터만 조회 하려면 결국 검색 조건이 포함된 SQL을 사용해야 한다. (모든 데이터를 불러와 엔티티 객체로 변경한 다음 검색하기 불가능 하기 때문에)  
-> JPA는 **SQL을 추상화한 JPQL)이라는 객체지향 쿼리 언어**를 제공해 이런 문제를 해결 한다. (SQL과 문법 유사)

- **JPQL은 엔티티 객체를 대상으로 쿼리**
- SQL은 데이터베이스 테이블 대상으로 쿼리
