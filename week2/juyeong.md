
### JPA 구현체로 하이버네이트를 사용하기 위한 핵심 라이브러리
hibernate-core : 하이버네이트 라이브러리
hibernate-jpa-2.1-api : JPA 2.1 표준 API를 모아둔 라이브러리
hibernate-entitymanager : 하이버네이트가 JPA 구현체로 동작하도록 JPA 표준을 구현한 라이브러리
hibernate-core와 hibernate-jpa-2.1-api를 함께 내려받음
 

### 매핑 어노테이션
@Entity :	이 클래스를 테이블과 매핑한다고 JPA에 알려준다. @Entity가 사용된 클래스=엔티티 클래스
@Table :	엔티티 클래스에 매핑할 테이블 정보를 알려줌
@Id  : 	필드를 Primary Key에 매핑, @Id가 사용된 필드=식별자 필드
@Column	: 필드를 컬럼에 매핑함.
매핑 정보가 없는 필드: 매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑
 

JPA를 실행하기 위한 기본 설정 파일 : persistence.xml
스프링 부트가 내부에서 JPA를 만들 때 persistence.xml 없이도 동작하도록 구현되어 있음
**스프링 부트를 사용한다면 persistence.xml이 없어도 된다.** 스프링과 JPA를 함께 사용한다면, application.yml을 사용
JPA 설정은 영속성 유닛(persistence-unit)부터 시작. 일반적으로 연결할 데이터베이스당 하나의 영속성 유닛 등록

### JPA 표준 속성 (특정 구현체에 종속되지 않음)
javax.persistence.jdbc.driver : JDBC 드라이버
javax.persistence.jdbc.user : 데이터베이스 접속 아이디
javax.persistence.jdbc.password : 데이터베이스 접속 비밀번호
javax.persistence.jdbc.url : 데이터베이스 접속 URL
### 하이버네이트 속성
hibernate.dialect : 데이터베이스 방언 설정
hibernate.show_sql : 하이버네이트가 실행한 SQL 출력
hibernate.format_sql : 하이버네이트가 실행한 SQL을 보기 좋게 정렬
hibernate.use_sql_comments : 쿼리를 출력할 때 주석도 함께 출력
hibernate.id.new_generator_mappings : JPA 표준에 맞춘 새로운 키 생성 전략 사용
 

### 방언(dialect) 
-> SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능
대부분의 JPA 구현체들은 다양한 데이터베이스 방언 클래스를 제공한다.
개발자는 JPA가 제공하는 표준 문법에 맞추어 JPA를 사용하면 되고, 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 알아서 처리해줌


### 애플리케이션 개발
- 엔티티 매니저 설정 
 1) Persistence 클래스가 persistence.xml 설정 정보를 읽어서 EntityManagerFactory 클래스 생성 
    - JPA를 동작시키기 위한 기반 객체
    - JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성
    - 생성 비용이 아주 크므로 애플리케이션 전체에서 딱 한 번만 생성하고 애플리케이션 전체에서 공유
    - 여러 스레드가 동시에 접근해도 안전함
2) EntityManagerFactory 클래스에서 EntityManager 생성
    - JPA의 기능 대부분을 EntityManager가 제공 (엔티티를 데이터베이스에 등록/수정/삭제/조회)
    - 내부에 데이터소스(데이터베이스 커넥션)을 유지하면서 데이터베이스와 통신
    - 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용 금지
        : 여러 스레드가 동시에 접근하면 동시성 문제 발생
3) 종료
    - EntityManager의 사용이 끝나면 EntityManager 종료
    - 애플리케이션을 종료할 때 EntityManagerFactory도 종료
4) 트랜잭션 관리
    - JPA 사용시 항상 트랜잭션 안에서 데이터 변경
    - EntityManager로부터 EntityTransaction을 받아옴
    - 트랜잭션 API를 사용해서 start() → 정상 동작하면 commit(), 예외 발생시 rollback()
5) 비즈니스 로직
EntityManager가 등록, 수정, 삭제, 조회 작업
등록 : persist()
수정 : update() 메서드 없음. setAge(20)처럼 엔티티 값만 변경하면 됨
삭제 : remove()
조회 : find() - 엔티티 타입과 기본키로 조회
 

### JPQL (Java Persistence Query Language)
JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 함
모든 데이터를 애플리케이션으로 불러와서 엔티티 객체로 변경한 다음 검색하는 것은 불가능
JPQL = 객체지향 쿼리 언어 
SQL과 사용 방법 거의 유사.
JPQL : 엔티티 객체를 대상으로 쿼리 (쉽게 이야기하면, 클래스와 필드를 대상으로 쿼리한다. JPQL은 데이터베이스 테이블을 전혀 알지 못한다.)
SQL :  데이터베이스 테이블을 대상으로 쿼리
