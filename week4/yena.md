# 4. 엔티티 매핑

JPA 사용시 가장 중요한 것은 엔티티와 테이블을 정확히 매핑하는 것...!

## @Entity

테이블과 매핑할 클래스에 사용. 엔티티 어노테이션이 붙은 클래스는 엔티티라 부른다. _(@Table과 무슨 차이인지 헷갈렸는데 JPA를 사용하는 클래스 (엔티티)임을 명시하는 의미로 생각됨)_

- 기본 생성자 필수
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없음
- 저장할 필드에 final 사용 X

## @Table

엔티티와 매핑할 테이블 지정. 생략 시 엔티티 이름을 테이블 이름으로 사용.

#### 속성

- uniqueConstraints: 스키마 자동 생성 기능으로 DDL 생성 시 유니크 제약조건만듦. 스키마 자동 생성 기능을 사용해 DDL을 만들 때만 사용.

## 다양한 매핑 사용

```java
@Entity
@Table(name="MEMBER")
pulbic class Member {
  ...

  @Enumerate(EnumType.STRING) //1
  private RoleType roleType;

  @Temporal(TemporalType.TIMESTAMP) //2
  private Date createdDate;

  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;

  @Lob //3
  private String description;

  ...
}

package jpabook.start;

public enum RoleType {
  ADMIN, USER
}
```

1. @Enumerated: 자바의 enum 사용.
2. @Teamporal: 자바의 날짜 타입 사용.
3. @Lob: DB의 clob, blob 타입 매핑할 수 있음.

## 데이터베이스 스키마 자동 생성

JPA는 매핑정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 자동 생성하는 기능을 지원한다.

스키마 자동 생성 기능이 만든 DDL은 운영 환경에서 사용할 만큼 완벽하지 않기 때문에 **개발 환경에서 사용하거나 매핑을 어떻게 해야 하는지 참고하는 정도로만 사용**하는 것이 좋다.

#### persistence.xml 속성 추가

`<property name="hibernate.hbm2ddl.auto" value="creaste" />`

이 속성을 추가하면 애플리케이션 실행 시점에 DB 테이블을 자동으로 생성한다.

> `hibernate.show_sql = true` 콘솔에 실행되는 테이블 생성 DDL 출력

## DDL 생성 기능

#### @Column 매핑 정보 속성

```java
@Column(name = "NAME", nullable = false, length = 10)
private String unserName;
```

- nullable: false로 지정하면 자동 생성되는 DDL에 not null 제약 조건 추가.
- length: 자동 생성되는 DDL에 문자 크기 지정

#### 유니크 제약조건 - @Table의 uniqueConstraints 속성

```java
@Entity
@Table(uniqueConstraints(
  name = "NAME_AGE_UNIQUE",
  columnNames = ("NAME", "AGE")))
public class Member {
  ...
}
```

이런 기능들은 DDL을 자동 생성할 때만 이용되고 JPA의 실행 로직에는 영향을 주지 않는다.
-> 개발자가 엔티티만 보고 다양한 제약조건 파악 가능

## 기본 키 매핑

기본 키를 애플리케이션에서 직접 할당하는 대신 **데이터베이스가 생성해주는 값**을 사용 하는 경우.  
(오라클 - 시퀀스 오브젝트, MySQL - AUTO_INCREMENT 등)

#### JPA 데이터베이스 기본 키 생성 전략

- 직접 할당
- 자동 생성: 대리 키 사용 방식
  - IDENTITY
  - SEQUENCE
  - TABLE

#### 기본 키 직접 할당 전략

`em.persist()`로 엔티티 저장 전 애플리케이션에서 기본 키를 직접 할당하는 방법.

- `@Id`로 매핑
  - 적용 가능 자바 타입
    - 자바 기본형
    - 자바 래퍼(Wrapper)형
    - String
    - java.util.Date
    - java.sql.Date
    - java.math.BigDecimal
    - java.math.BingInteger

#### IDENTITY 전략

기본 키 생성을 DB에 위임. DB에 값을 저장하고 나서 기본키 값을 구할 수 있을 때 사용.

`em.persist()` 호출해 엔티티를 저장한 후 식별자 값을 획득한다. 식별자 값을 가져오고 나서 영속성 컨텍스트에 저장.

- `@GneratedValue(strategy = GenerationType.IDENTITY)`: 기본키 값을 얻기 위해 JPA가 **DB 추가 조회**.

주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용.

> MySQL의 AUTO_INCREMENT 기능
>
> ```sql
> CREATE TABLE BOARD (
>   ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
>   DATA VARCHAR(255)
> );
>
> INSERT INTO BOARD(DATA) VALUES('A');
> INSERT INTO BOARD(DATA) VALUES('B');
> ```

**엔티티가 영속 상태 되려면 식별자가 반드시 필요(엔티티 식별자 값으로 구분)**  
-> IDENTITY 식별자 생성 전략은 엔티티를 저장해야 식별자 값을 얻을 수 있음  
-> em.persist()호출 시 insert sql을 DB로 전달  
=> **트랜잭션을 지원하는 쓰기 지연 동작X**

#### SEQUENCE 전략

DB 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트이다.

`em.persist()`를 호출할 때 먼저 DB 시퀀스를 사용해서 식별자를 조회. **조회한 식별자를 엔티티에 할당 후 엔티티를 영속성 컨텍스트에 저장**. 이후 트랜잭션 커밋해 플러시가 일어나면 엔티티를 DB에 저장.

오라클, PostgreSQL, DB2, H2에서 사용.

1. DB에서 시퀀스 생성 DDL 실행
2. 엔티티에 데이터베이스 시퀀스 매핑

   - `@SequenceGenerator`
   - `name = "시퀀스 생성기 이름"` 시퀀스 생성기 등록
   - `sequenceName = "DB 시퀀스 이름"` 속성 이름으로 시퀀스 지정. JPA가 시퀀스 생성기를 실제 DB의 시퀀스와 매핑.

3. 식별자에 키 생성 전략 설정, 시퀀스 생성기 선택
   - `@GeneratedValue`의 속성 설정
   - `strategy = GenerationType.SEQUENCE`
   - `generator = "시퀀스 생성기 이름"`

#### @SequenceGenerator의 allocationsize 속성

시퀀스 한 번 호출에 증가하는 수 이며 기본값은 50이다.

- 기본값이 50
- JPA가 기본으로 생성하는 DB 시퀀스  
   `create sequence [sequenceName] start with 1 increment by 50`
- 시퀀스 호출할 때마다 값이 50씩 증가
- DB 시퀀스 값이 하나씩 증가 하는 설정이라면 1로 변경

**성능 최적화**에 사용한다.

- **DB 접근 횟수 줄임**  
   SEQEUNCE 전략은 DB에 두번 접근(SELECT, INSERT).  
   설정값 만큼 한 번에 시퀀스 증가 시키고 메모리에 할당.  
   메모리에서 식별자 값을 할당.
- **기본키 값 충돌 방지**  
   시퀀스 값을 먼저 선점. 여러 JVM이 동시 동작해도 값 충돌X.
- DB에 직접 접근하므로 데이터 등록시 시퀀스 값이 한번에 많이 증가  
   INSERT 성능 중요치 않으면 값을 1로 설정.

#### TABLE 전략

키 생성 전용 테이블을 만들어 여기에 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스를 흉내내는 전략. 테이블을 사용하므로 모든 DB에 적용 가능.

`em.persist()`호출 시 DB 시퀀스 생성용 테이블에서 식별자 값 획득 후 영속성 컨텍스트에 저장.

1. 키 생성 전용 테이블 생성
   ```SQL
   create table MY_SEQUENCES (
     sequecne_name varchar(255) not null,
     next_val bigint,
     primary key (sequence_name)
   )
   ```
2. 엔티티에 키 생성 전용 테이블 매핑
   - `@TableGenerator`
   - `name = "테이블 키 생성기 이름"` 테이블 키 생성기 등록
   - `TABLE = "MY_SEQUENCES"` 테이블을 키 생성용 테이블로 매핑
   - `pkColumnValue = "키로 사용할 값(엔티티 이름이 기본 값)"`
   - `allocationSize = 1` 기본값 50
3. 식별자에 테이블 키 생성기 매핑
   - `@GeneratedValue`
   - `strategy = GenerationType.TABLE`
   - `generator = "테이블 키 생성기"`
4. `pkColumnValue`으로 지정한 값이 MY_SEQUENCES 테이블에 컬럼으로 추가, 키 생성기 사용시 next_val 컬럼 값이 증가

#### @TableGenerator의 allocationsize 속성

시퀀스 한 번 호출에 증가하는 수. 기본 값 50.

성능 최적화에 사용한다. (SEQUENCE 전략과 최적화 방법 동일)

- **DB 접근 횟수 줄임**  
  TABLE 전략은 값을 조회 하면서 SELECT 사용. 다음 값 증가 위해 UPDATE 사용.

#### AUTO 전략

DB 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.  
선택될 전략 따라 시퀀스나 키 생성용 테이블을 미리 만들어두어야 한다.

`@GeneratedValue(strategy = GenerationType.AUTO)`

#### 권장하는 식별자 선택 전략

테이블 기본 키 선택 전략

- 자연 키(natural key): 비즈니스에 의미 있는 키(주민번호, 이메일, 전화번호)
- 대리 키(surrogate key), 대체 키: 비즈니스와 관련 없는 임의로 만들어진 키(오라클 시퀀스, auto_increment, 키생성 테이블 사용)

자연키도 변할 수 있고, 비즈니스의 환경(요구사항)은 언젠가 변한다...!  
=> 자연키의 후보가 되는 컬럼들은 필요에 따라 유니크 인덱스를 설정해서 사용  
**=> 자연키 보다 대리키 권장**

## 필드와 컬럼 매핑

JPA가 제공하는 필드와 컬럼 매핑용 어노테이션들은 사용할 일이 있을 때 찾아서 자세히 알아보기!

- @Column: 컬럼 매핑
- @Enumerated: 자바 enum 타입 매핑
- @Temporal: 날짜 타입 매핑
- @Lob: BLOB, CLOB 타입 매핑
- @Transient: 특정 필드 DB에 매핑 하지 않는다.
- @Access: JPA가 엔티티에 접근하는 방식 지정
