# 5. 연관관계 매핑 기초

**객체**는 **참조(주소)** 사용해 연관관계 맺는다.  
**테이블**은 **외래키**를 사용해 연관관계를 맺는다.

ORM에서 **객체의 참조**와 **테이블의 외래키**를 **매핑**하는 부분이 중요.

(+회원과 팀 예제로 배우기)

## 단방향 연관관계

#### 다대일(N:1) 단방향 관계 매핑

- 객체 연관관계
  - 회원 객체는 Member.team 필드로 팀 객체와 연관관계
  - 회원 -> 팀 객체는 단방향 연관관계. (팀->회원 알 수 없음)
- 테이블 연관관계
  - 회원 테이블은 TEAM_ID 외래키로 팀 테이블로 연관관계
  - 회원 <-> 팀 테이블은 양방향 관계. (외래키 하나로 양방향 조인 가능)
- 객체 연관관계와 테이블 연관관계 차이점
  - **참조를 통한 연관관계는 언제나 단방향**이다. 양방향으로 만들려면 양쪽에 필드를 추가해 참조를 보관해야 한다.  
    **-> 양방향이 아닌 단방향 관계 2개**

#### 객체 연관관계 - 객체 그래프 탐색

객체가 참조를 사용해서 연관관계를 탐색하는 것이 객체 그래프 탐색이다.  
`Team findTeam = member1.getTeam();`

#### 데이터베이스 테이블 연관관계 - 조인

데이터베이스는 외래 키를 사용해서 연관관계를 탐색하는데 이것을 조인이라 한다.

```sql
SELECT T.*
FROM MEMBER M
   JOIN TEAM T ON M.TEAM_ID T.TEAM_ID
WHERE M.MEBER_ID = 'member1'
```

#### 객체 관계 매핑

객체에 다른 엔티티를 매핑하기.

- @ManyTonOne  
  다대일(N:1)관계 매핑 정보.
- @JoinColumn(name="TEAM_ID")  
  조인 컬럼은 외래 키를 매핑. name 속성에 매핑할 외래 키 이름을 지정. 생략 가능.

#### 일대일 관계

단방향 관계를 매핑할 때 다대일과 일대일(@OneToOne) 둘 중 어떤 것을 사용할지는 반대편 관계에 달려 있다. 반대편이 일대다 관계면 다대일 사용, 반대편이 일대일 관계면 일대일 사용.

## 연관관계 사용

#### 저장

연관관계 매핑한 엔티티 저장.

```java
member1.setTeam(team1); // 회원->팀 참조
em.persist(member1); //저장
```

JPA는 참조한 팀의 식별자(Team.id)를 외래키로 사용해 적절한 등록 쿼리를 생성한다.

##### 조회

연관관계가 있는 엔티티를 조회 하는 방법 두 가지.

1. 객체 그래프 탐색

   ```java
   Member member = em.find(Member.class, "member1");
   Team team = member.getTeam(); //객체 그래프 탐색
   System.out.println("팀 이름 = " + team.getName());
   ```

2. 객체지향 쿼리 사용 JPQL

   회원을 대상으로 팀1에 소속된 회원만 조회.
   -> 회원과 연관된 팀 엔티티를 검색 조건으로 사용.  
   JPQL도 SQL처럼 조인을 지원.

   ```java
   String jpql = "select m from Member m join m.team t where " + "t.name=:teamName";

   List<Member> resultList = em.createQuery(jpql, Member,class)
      .setParameter("teamName", "팀1")
      .getResultList();

    for (Member memeber : resultList) {
      System.out.println("[query] member.username=" + member.getUsername());
    }
   ```

   jpql을 보면 회원이 팀과 관계를 가지고 있는 필드(m.team)을 사용해 Member와 Team을 조인. where절 검색조건으로 조인한 t.name을 사용.  
   `:teamName`의 `:`은 파라미터를 바인딩 받는 문법.

#### 수정

연관관계 수정.

수정은 em.update()같은 메소드가 따로 없고 불러온 엔티티의 값만 변경하면 트랜잭션 커밋 시 플러시가 일어나 변경 감지 기능이 작동한다. 그리고 변경사항을 데이터베이스에 자동으로 반영한다.

연관관계 수정도 참조 대상만 변경하면 JPA가 자동으로 처리한다.

#### 삭제

연관관계 제거.

회원1을 팀에 소속하지 않도록 변경.

```java
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null); //연관관계 제거
```

연관관계를 null로 설정. jpa가 update문을 실행함.

#### 연관된 엔티티 삭제

연관된 엔티티를 삭제 하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 외래 키 제약조건으로 인한 오류가 발생하지 않는다.

```java
member1.setTeam(null); //회원1 연관관계 제거
member2.setTeam(null); //회원2 연관관계 제거
em.remove(team); //팀 삭제
```

## 양방향 연관관계

회원->팀 접근, 팀->회원 접근 양방향 연관관계 매핑.

데이터베이스 테이블은 외래키 하나로 양방향 조회가 가능.

> JPA는 List 포함한 Collection, Set, Map 같은 다양한 컬렉션 지원

#### 일대다 관계 연관관계 매핑

일대다 관계에서 객체 그래프 탐색을 위해 엔티티에 컬렉션 필드를 추가한다. (팀->회원 일대다 관계)

`List<Member> members`

그리고 일대다 관계를 매핑하기 위해 @OneToMany 매핑 정보를 사용한다.  
mappedBy 속성으로 양방향 매핑일때 반대쪽 매핑의 필드 이름을 값으로 사용한다. (Member.team)

`@OneToMany(mappedBy = "team")`

#### 일대다 컬렉션 조회

팀->회원 컬렉션으로 객체그래프탐색.

```java
Team team = em.find(Team.class, "team1");

List<Member> members = team.getMembers(); // 팀->회원 객체그래프 탐색
```

## 연관관계의 주인

테이블은 외래 키 하나로 두 테이블의 연관관계를 관리.

객체는 참조를 통해 연관관계 -> 참조를 통한 연관관계는 항상 단방향-> 엔티티를 양방향으로 매핑하면 단방향 2개를 관리.

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나.

이런 차이로 JPA는 두 객체 연관관계 중 하나를 주인으로 정해 테이블의 외래키를 관리한다.

=> 연관관계의 주인

#### 양방향 매핑 규칙

양방향 매핑 시 두 연관관계 중 하나를 주인으로 정한다. **연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)할 수 있다.** 주인이 아닌 쪽은 읽기만 할 수 있다.

연관관계의 주인은 **외래 키 관리자**를 선택하는 것.

- @OneToMany의 mappedBy: 속성 값으로 연관관계 주인을 지정.

#### 연관관계 주인은 외래 키가 있는 곳

외래 키가 없는 테이블을 주인으로 선택하면 물리적으로 다른 테이블의 외래 키를 관리해야 하기 때문에 관리해야 할 외래 키가 있는 테이블(엔티티)를 주인으로 선택해 **자기 테이블의 외래키를 관리**하도록 한다.

## 양방향 연관관계 저장

양방향 연관관계를 사용해 팀1, 회원1, 회원2 저장.

```java
//팀1 저장
Team team1 = new Team("team1", "팀1");
em.persist(team1);

//회원1 저장
Member member1 = new Member("member1", "회원1");
member1.setTeam(team1); //연관관계 설정 member1 -> team1
em.persist(member1);

//회원2 저장
Member member2 = new Member("member2", "회원2");
member2.setTeam(team1); //연관관계 설정 member2 -> team1
em.persist(member2);
```

위처럼 저장하면 MEMBER 테이블의 외래 키(TEAM_ID)에 팀의 기본 키 값이 저장되어 있다.

양방향 연관관계는 연관관계의 주인이 외래키를 관리한다. 따라서 **주인이 아닌 방향은 값을 설정하지 않아도 DB에 외래키 값이 정상 입력된다.**

**주인이 아닌 곳에 입력된 값**은 외래 키에 영향을 주지 않아서 **DB에 저장될 때 무시**된다.

## 양방향 연관관계의 주의점

연관관계 주인에는 값을 입력하지 않고, 주인 아닌 곳에만 값 입력하는 것을 주의하기!!!! (DB에 외래키 값이 정상적으로 저장되지 않은 경우 의심해보기)

연관관계의 주인만 외래 키 값을 변경할 수 있다.

회원 -> 팀 연관관계에서 회원이 주인일 때 Team.members에만 값을 저장하고 Member.team에는 아무 값도 입력하지 않으면 외래키에 null 값이 저장됨.

#### 순수한 객체까지 고려한 양방향 연관관계

객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.

양방향 연관관계이면 두 객체에 각각 양방향의 관계(참조)를 설정해 순수한 객체 상태에서도 동작하도록 하는 것이 안전하다. _(한 쪽만 연관관계 설정시 -> 조회 값 null)_

#### 연관관계 편의 메소드

양방향 연관관계를 설정하는 코드를 하나인 것처럼 묶어서 한 메소드로 양방향 관계를 모두 설정하도록 하는 코드이다.

기존 코드

```java
member.setTeam(team);
team.getMembers().add(member);
```

리팩토링

```java
public class Member {

  private Team team;

  public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
  }
}
```

#### 연관관계 편의 메소드 작성시 주의사항

연관관계를 변경할 때는 기존 관계를 모두 제거 후 변경하도록 기존 관계를 삭제하는 코드를 꼭 작성해야 한다.

```java
public void setTeam(Team team) {

  //기존 팀과 관계를 제거
  if (this.team != null) {
    this.team.getMembers().remove(this);
  }

  this.team = team;
  team.getMembers().add(this);
}
```

## 정리

단방향 매핑과 비교해 양방향 매핑은 복잡하다. 객체에서 서로 다른 단방향 연관관계 2개를 양방향 처럼 보이게 하려고 많은 수고가 필요하다. 반면 관계형 데이터베이스에서는 외래키 하나로 단순히 관리할 수 있다.

단방향과 비교해 양방향의 장점은 반대방향으로 객체그래프탐색 기능이 추가된 것뿐이다.

비즈니스 로직에 따라 반대 방향 객체그래프탐색이 필요할 때만 양방향을 사용하도록 코드를 추가해도 된다.

#### 연관관계 주인 정하는 기준

비즈니스 중요도가 아닌 외래키의 위치가 있는곳을 연관관계 주인으로 해야함!
