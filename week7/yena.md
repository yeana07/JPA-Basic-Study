# 7. 고급 매핑

## 상속 관계 매핑

관계형 데이터베이스는 객체지향 언어에서 다루는 상속이라는 개념이 없다. 대신 **슈퍼타입 서브타입 관계(Super-Type Sub-Type Relationship)** 모델링 기법이 객체의 상속 개념과 가장 유사하다.

ORM에서의 상속 관계 매핑은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

슈퍼타입 서브타입 관계를 물리 모델인 테이블로 구현하는 3가지 방법.

1. 각각의 테이블로 변환(JPA의 조인 전략)
2. 통합 테이블로 변환(JPA의 단일 테이블 전략)
3. 서브타입 테이블로 변환(JPA의 구현 클래스마다 테이블 전략)

### 조인 전략 Joined Strategy

엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아 기본키 + 외래키로 사용하는 전략이다.

조회할 때 조인을 자주 사용한다.

**주의할 점**: 객체는 타입으로 구분할 수 있지만, 테이블은 타입의 개념이 없다. 따라서 타입을 구분하는 컬럼을 추가해야 한다. (부모 테이블에 `DTYPE` 구분 컬럼 사용)

#### 매핑 정보

- `@Inheritance(strategy = InheritanceType.JOINED)`: 부모 클래스에 사용하는 상속 매핑. 속성 값으로 매핑 전략을 지정.
- `@DiscriminatorColumn(name = "DTYPE")`: 부모 클래스에 구분 컬럼 지정. 기본값이 `DTYPE`이다.(name 속성 생략 가능)
- `@DiscriminatorValue("M")`: 엔티티를 저장할 때 구분 컬럼에 입력할 값 지정.
- `@PrimarykeyJoinColumn(name = "BOOK_ID")`: 필요시 자식 테이블에서 부모 테이블의 ID 컬럼명을 변경해서 사용. 기본값은 그대로 사용.

#### 장점

- 테이블이 정규화된다
- 외래 키 참조 무결성 제약조건을 활용할 수 있다
- 저장공간을 효율적으로 사용한다

#### 단점

- 조회할 때 조인이 많이 사용되어 성능이 저하될 수 있다
- 조회 쿼리가 복잡하다
- 데이터를 등록할 때 INSERT SQL을 두 번 실행한다

#### 특징

- JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 하이버네이트 포함한 구현체는 구분 컬럼(`@DiscriminatorColumn`) 없이도 동작한다.

### 단일 테이블 전략 Single-Table Strategy

테이블을 하나만 사용하고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분한다. 조회할 때 조인을 사용 않아 일반적으로 가장 빠르다.

**주의할 점**: 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.

#### 매핑 정보

- `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)`: 부모 테이블에 상속 매핑 전략을 단일 테이블 전략으로 지정.

#### 장점

단일 테이블 전략의 장단점은 하나의 테이블을 사용하는 특징과 관련이 있다.

- 조인이 필요 없으므로 조회 성능이 빠르다
- 조회 쿼리가 단순하다

#### 단점

- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다
- 단일 테이블에 모든것을 저장하므로 테이블이 커질 수 있다. 상황에 따라 조회 성능이 오히려 느릴 수 있다.

#### 특징

- 구분 컬럼을 꼭 사용해야 한다. `@DiscriminatorColumn`을 꼭 사용.
- `@DiscriminatorValue`를 지정 않으면 기본으로 엔티티 이름을 사용한다.

### 구현 클래스마다 테이블 전략 Table-per-Concrete-Class Strategy

자식 엔티티마다 테이블을 만든다. 자식 테이블 각각에 필요한 컬럼이 모두 있다. 일반적으로 추천하지 않는 전략이다. (DB 설계자, ORM 전문가 둘 다 추천X)

#### 매핑 정보

- `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`: 부모 테이블에 상속 매핑 전략을 단일 테이블 전략으로 지정.

#### 장점

- 서브 타입을 구분해서 처리할 때 효과적이다
- not null 제약조건을 사용할 수 있다

#### 단점

- 여러 자식 테이블을 함께 조회할 때 성능이 느리다(SQL의 UNION 사용)
- 자식 테이블을 통합해서 쿼리하기 어렵다

#### 특징

- 구분 컬럼을 사용하지 않는다

## @MappedSuperClass

부모 클래스는 테이블과 매핑하지 않고 상속 받는 자식 클래스에게 **매핑 정보만 제공**하고 싶으면 `@MappedSuperClass`를 사용한다.

추상 클래스와 비슷한데 실제 클래스와 매핑 되지 않고 단순히 매핑 정보를 상속할 목적으로만 사용된다.

엔티티가 공통으로 사용하는 **매핑 정보를 모아주는 역할**을 한다.

ORM의 진정한 상속 매핑은 객체 상속을 데이터베이스의 슈퍼타입 서브타입 관계와 매핑하는 것이다.

#### 관련 어노테이션

- `@AttributeOverride(s)`: 부모로부터 물려받은 매핑 정보 재정의(컬럼명 재정의)
- `@AssociationOverride(s)`: 연관관계 재정의

#### 특징

- 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용한다
- `@MappedSuperClass`로 지정한 클래스는 엔티티가 아니므로 `em.find()`나 JQPL을 사용할 수 없다
- 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로 추상 클래스로 만드는 것을 권장한다

## 복합 키와 식별 관계 매핑

### 식별 관계와 비식별 관계

DB 테이블 사이 관계는 외래 키가 기본 키에 포함되는지 여부에 따라 식별 관계와 비식별 관계로 구분한다.

#### 식별 관계 Identifying Relationship

식별 관계는 부모 테이블의 기본 키를 내려받아 자식 테이블의 기본키+외래키로 사용하는 관계다.

#### 비식별 관계 Non-Identifying Relationship

비식별 관계는 부모 테이블의 기본 키를 받아 자식 테이블의 외래 키로만 사용하는 관계다.

비식별 관계는 외래 키에 NULL 허용 여부에 따라 필수적 비식별 관계와 선택적 비식별 관계로 나뉜다.

- **필수적 비식별 관계(Mandatory)**: 외래키에 NULL 허용 않음. 연관관계를 필수적으로 맺음.
- **선택적 비식별 관계(Optional)**: 외래키에 NULL 허용. 연관관계를 맺을지 말지 선택 가능.

-> JPA는 둘 다 지원  
**=> 비식별 관계를 주로 사용하고 꼭 필요한 곳에만 식별 관계를 사용**

### 복합 키: 비식별 관계 매핑

기본 키 컬럼이 하나면 `@Id`로 단순히 매핑.

둘 이상의 컬럼으로 구성된, 식별키가 둘 이상인 복합 기본키를 JPA에서 사용 하려면 별도의 식별자 클래스를 만들어야 한다.

> JPA는 영속성 컨텍스트에 엔티티 보관 시 엔티티 식별자를 키로 사용한다. 그리고 식별자 구분 위해 `equals`와 `hashCode`를 사용해 동등성 비교를 한다.  
> 식별자 필드가 2개 이상이면 별도의 식별자 클래스를 만들고 그 클래스에 `equals`와 `hashCode`를 구현해야 한다.

JPA는 복합 키 지원 위해 `@IdClass`, `@EmbeddedId` 2가지 방법을 제공한다.

(+부모(PARENT) 자식(CHILD)은 '객체 상속'과는 무관한 예제)

#### `@IdClass`

관계형 데이터베이스에 가까운 방식. `@IdClass`로 복합키의 식별자 클래스를 지정한다.

#### `@IdClass`를 사용하는 클래스 작성

PARENT 테이블의 기본키를 PARENT_ID1, PARENT_ID2로 묶은 복합키로 구성. CHILD 테이블은 PARENT의 복합키를 내려받는다.

**PARENT 클래스**

```java
@Entity
@IdClass(ParentId.class)
public class Parent {

  @Id
  @Column(name = "PARENT_ID1")
  private String id1; //ParentId.id1과 연결

  @Id
  @Column(name = "PARENT_ID2")
  private String id2; //ParentId.id2와 연결

  ...
}
```

**식별자 클래스**

복합키를 매핑하는 별도의 식별자 클래스

```java
public class ParentId implements Serializable {

  private String id1; //Parent.id1 매핑
  private String id2; //Parent.id2 매핑

  public ParentId() {
  }

  public ParentId(String id1, String id2) {
    this.id1 = id1;
    this.id2 = id2;
  }

  @Override
  public boolean equals(Object o) {...}

  @Override
  public int hashCode() {...}
}
```

**Child 클래스**

```java
@Entity
public class Child {

  @Id
  private String id;

  @ManyToOne
  @JoinColumns({
    @JoinColumn(name = "PARENT_ID1",
      referencedColumnName = "PARENT_ID1"),
    @JoinColumn(name = "PARENT_ID2",
      referencedColumnName = "PARENT_ID2"),
  })
  private Parent parent;
}
```

부모 클래스의 기본 키 컬럼이 복합 키이므로 자식 테이블의 외래 키도 복합키다. 외래키 매핑 시 여러 컬럼 매핑 위해 `@JoinColumns`사용.

`@JoinColumn`의 `name`속성과 `referencedColumnName`속성 값이 같으면 `referencedColumnName`는 생략 가능.

#### `@IdClass`사용 시 식별자 클래스 조건

- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다
- `Serializable` 인터페이스를 구현해야 한다
- 기본 생성자가 있어야 한다
- 식별자 클래스는 public 이어야 한다

#### `@IdClass`로 지정한 복합키 사용하기

**저장**: 매핑한 복합키를 사용해 저장하면 영속성 컨텍스트에 엔티티를 등록하기 직전에 **내부에서** 식별자값이 될 필드 값(Parent.id1, Parent.id2)을 사용해 **식별자 클래스(ParentId)를 생성**하고 영속성 컨텍스트의 키로 사용한다.

`parent.setId1("myId1")`  
`parent.setId1("myId2")`  
...  
`em.persist(parent);`

**조회**: 식별자 클래스를 사용해 엔티티를 조회한다.

`ParentId parentId = new ParentId("myId1", "myId2")`  
`Parent parent = em.find(Parent.class, parentId)`

#### `@EmbeddedId`

객체지향에 가까운 방법.

#### `@EmbeddedId`를 사용하는 클래스 작성

**PARENT 클래스**

```java
@Entity
public class Parent {

  @EmbeddedId
  private ParentId id;

  private String name;
  ...
}
```

Parent 엔티티에서 식별자 클래스를 직접 사용하고 `@EmbeddedId`를 적으면 된다.

**식별자 클래스**

```java
@Embeddable
public class ParentId implements Serializable {

  @Column(name = "PARENT_ID1")
  private String id1;
  @Column(name = "PARENT_ID2")
  private String id2;

  //생성자
  ...

  //equals and hashCode 구현
  ...
}
```

#### `@EmbeddedId` 적용한 식별자 클래스 조건

- `@Embeddable` 어노테이션을 붙여주어야 한다
- `Serializable` 인터페이스를 구현해야 한다
- `equals`, `hashCode`를 구현해야 한다
- 기본 생성자가 있어야 한다
- 식별자 클래스는 public이어야 한다

#### `@EmbeddedId`로 지정한 복합키 사용하기

**저장**: **식별자 클래스를 직접 생성**해서 사용한다.

`ParentId parentId = new ParentId("myId1", "myId2")`  
`parent.setId(parentId)`  
...  
`em.persist(parent)`

**조회**: 식별자 클래스를 직접 사용해 조회한다._(`@IdClass`와 동일)_

`ParentId parentId = new ParentId("myId1", "myId2")`  
`Parent parent = em.find(Parent.class, parentId)`

#### 복합 키와 equals(), hashCode()

복합 키는 `equals()`, `hashCode()`를 필수로 구현해야 한다.

> 기본 `equals()`는 인스턴스 참조 값을 비교

영속성 컨텍스트는 엔티티 식별자를 키로 사용해 엔티티를 관리한다.

식별자를 비교하는 `equals()`, `hashCode()`가 적절히 구현되어 있지 않으면 영속성 컨텍스트의 엔티티 관리에 문제가 발생한다.  
-> 잘못된 엔티티 조회, 엔티티 못 찾음...

#### `@IdClass` vs `@EmbeddedId`

둘 중 선택해서 일관성있게 사용하면 된다.

`@EmbeddedId`가 `@IdClass`와 비교해 더 객체지향적이고 중복도 없지만, 특정 상황에는 JPQL이 더 길어질 수 있다.

### 복합 키: 식별 관계 매핑

식별 관계에서 자식 테이블은 부모 테이블의 기본 키를 포함해 기본 키를 구성해야 하므로 `@Idclass`나 `@EmbeddedId`를 사용해 식별자를 매핑한다.

부모, 자식, 손자까지 계속 기본 키를 전달한다.

#### `@IdClass`와 식별 관계

식별 관계는 기본 키와 외래 키를 같이 매핑. -> 식별자 매핑 `@Id`와 연관관계 매핑 `@ManyToOne`같이 사용

#### `@EmbeddedId`와 식별 관계

`@EmbeddedId`로 식별 관계를 구성할 때는 식별 관계로 사용할 연관관계의 속성에 `@Id`대신에 `@MapsId`를 사용해야 한다. _(부모의 PK 컬럼을 ID로 지정)_

`@MapsId`는 **외래키와 매핑한 연관관계를 기본 키에도 매핑**하겠다는 뜻이다. 속성 값은 `@EmbeddedId`를 사용한 식별자 클래스의 기본 키 필드를 지정.

**Child 클래스**

```java
@Entity
public class Child {

  @EmbeddedId
  private ChildId id;

  @MapsId("parentId") //ChildId.parentId 매핑
  @ManyToOne
  @JoinColumn(name = "PARENT_ID")
  public Parent parent;

  private String name;
  ...
}
```

**Child Id 클래스**

```java
@Embeddable
public class ChildId implements Serializable {

  private String parentId; //@MapsId("parentId")로 매핑

  @Column(name = "CHILD_ID")
  private String id;

  //equals, hashCode
  ...
}
```

### 비식별 관계로 구현

매핑도 쉽고 코드도 단순하다. 복합 키가 없기 때문에 별도의 복합 키 클래스를 만들지 않아도 된다.

### 일대일 식별 관계

일대일 식별 관계는 자식 테이블의 기본 키 값으로 부모 테이블의 기본 키 값을 사용한다. 그래서 부모 테이블의 기본 키가 복합 키X -> 자식 테이블 기본 키도 복합 키 구성X.

식별자가 단순히 컬럼 하나면 `@MapsId`를 사용하고 속성 값은 비워두면 된다. _(`@MapsId`는 `@Id`를 사용해 식별자로 지정한 필드와 매핑)_

**부모**

```java
@Entity
public class Board {

  @Id @GeneratedValue
  @Column(name = "BOARD_ID")
  private Long id;

  private String title;

  @OneToOne(mappedBy = "board")
  private BoardDetail boardDetail;
  ...
}
```

**자식**

```java
@Entity
public class BoardDetail {

  @Id
  private Long boardId;

  @MapsId //BoardDetail.boardId와 매핑
  @OneToOne
  @joinColumn(name = "BOARD_ID")
  private Board board;

  private String content;
  ...
}
```

### 식별, 비식별 관계의 장단점

데이터베이스 설계 관점에서 식별 관계보다 비식별 관계를 선호한다.

- 식별 관계는 자식 테이블의 기본 키 컬럼이 점점 늘어난다. 조인할 때 SQL이 복잡해지고 기본 키 인덱스가 불필요하게 커질 수 있다.
- 식별 관계는 복합 기본 키를 만들어야 하는 경우가 많다.
- 식별 관계에서 기본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많다._(자연키는 변경 가능성 높음)_ 반면 비식별 관계의 기본 키는 대리 키를 주로 사용한다.
- 식별 관계는 부모 테이블의 기본 키를 자식 테이블의 기본 키로 사용하므로 테이블 구조가 유연하지 못하다.

객체 관계 매핑의 관점에서 보는 비식별 관계를 선호하는 이유.

- 일대일 관계를 제외하고 식별 관계는 복합 기본 키를 사용한다. JPA에서는 별도의 복합 키 클래스가 필요하다.
- 비식별 관계의 기본 키는 주로 대리 키를 사용하는데 JPA는 `@GeneratedValue`와 같은 대리키 생성을 위한 편리한 방법을 제공한다.

식별 관계의 장점

- 기본 키 인덱스를 활용하기 좋다
- 상위 테이블 기본 키 컬럼을 자식, 손자 테이블이 가지고 있으므로 특정 상황에 조인 없이 하위 테이블만으로 검색을 완료할 수 있다.

**=> 가능하면 비식별 관계 사용, 기본 키는 Long 타입의 대리 키 사용하기**  
**=> 선택적 비식별 관계보다 필수적 비식별 관계(내부조인 사용 가능) 사용하기**

## 조인 테이블

데이터베이스 테이블 연관관계 설계하는 방법

- 조인 컬럼 사용(외래키)
- 조인 테이블 사용(테이블 사용)

선택적 비식별 관계는 외래 키에 null을 허용하므로 조인할 때 외부 조인을 사용한다.(내부 조인하면 모든 데이터가 나오지 않고, 외래 키 값 대부분이 null이 되는 경우가 있을 수 있음...)

조인 테이블은 두 테이블의 외래 키를 가지고 연관관계를 관리한다. (필요할 때에 조인테이블에 값을 추가)

조인 테이블을 사용하면 관리할 테이블이 늘어나고 연관된 두 테이블을 조인하려면 조인 테이블까지 추가로 조인해야 한다.

**=> 기본은 조인 컬럼을 사용하고 필요시 조인 테이블 사용**

- 객체와 테이블을 매핑할 때 조인 컬럼은 `@joinColumn`으로 매핑하고, 조인 테이블은 `@JoinTable`로 매핑한다.
- 조인 테이블은 주로 다대다 관계를 일대다, 다대다 관계로 풀어낼 때 사용한다.

#### `@JoinTable` 속성

- `name`: 매핑할 조인 테이블 이름
- `joinColumns`: 현재 엔티티를 참조하는 외래 키
- `inversJoinColumns`: 반대 방향 엔티티를 참조하는 외래 키

## 엔티티 하나에 여러 테이블 매핑

`@SecondaryTable`을 사용하면 한 엔티티에 여러 테이블을 매핑할 수 있다.  
-> 잘 사용하지 않는다.  
-> 테이블당 엔티티 각각 만들어 일대일 매핑하기 권장.

#### `@SecondaryTable` 속성

- `@SecondaryTable.name`: 매핑할 다른 테이블의 이름.
- `@SecondaryTable.pkJoinColumns`: 매핑할 다른 테이블의 기본 키 컬럼 속성.

필드에 테이블을 지정 않으면 기본 테이블에 매핑된다. 더 많은 테이블을 매핑하려면 `@SecondaryTables`를 사용.
