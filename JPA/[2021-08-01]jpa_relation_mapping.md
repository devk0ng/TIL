# 연관관계 매핑

## 연관관계 용어정리와 간단한 설명

연관관계 매핑을 알아보기에 앞서 기본적인 용어를 정리하고 들어간당

- 방향 - 단방향, 양방향
- 다중성 - 다대일, 일대다, 일대일, 다대다
- 연관관계의 주인 - 양방향 관계에서는 연관관계 주인이라는 것을 설정해줘야해!! 중요!!

<br/>

### 방향 별 사용방법

단뱡향은 그냥 @OneToMany, @ManyToOne ... 등 관계를 나타내는 어노테이션과 @JoinColumn(name = "blabla") 를 통해 외래키를 매핑해주고 그냥 사용하면 돼!!

but, **양방향의 경우에는 두 관계중 연관관계의 주인을 결정해 줘야해!!**

연관관계주인은 외래키를 관리하는 놈이라고 생각하면 편할 것 같애.

**연관관계의 주인만이 외래키를 등록, 수정할 수 있으며 주인이 아닌쪽은 읽기만 가능해**

주인이 아닌 쪽에 mappedBy 속성을 통해 주인을 지정해주면 돼.

🤔🤔🤔 그럼 누구를 주인으로 해야할까? 🤔🤔🤔
외래키가 있는 곳을 주인으로 정해!!

👏👏 양방향 연관관계에서 주의할 것!! 👏👏

- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자!! 연관관계 편의 메소드를 생성하면 좋아

  ```java
  public class Member {
  	private Team team;

  	public void setTeam(Team team) {
  		if(this.team != null) {
  			this.team.getMembers().remove(this);
  		}

  		this.team = team;
  		team.getMembers().add(this); //양방향 셋팅
  	}
  }
  ```

  이렇게 말이야!! if 조건문을 넣은 이유는 뭘까나??

  해당 부분을 체크하지 않을 경우

  ```java
  //if check문이 없다고 생각하고 아래 코드를 보면
  member.setTeam(teamA);
  member.setTeam(teamB);

  Member findMember = teamA.getMember();
  Member findMember = teamB.getMember();
  ```

  이러한 코드를 생각해 볼때 teamA에도 해당 member가, teamB에도 해당 member가 있는 것으로 나오게 되어버린다. 이러한 부분을 막기위해 추가해준 조건이다 이말이야.

간단히 정리하자면 단방향 매핑만으로 연관관계 매핑은 이미 끝났다고 볼 수 있기에 만약 프로그램에서 역으로 탐색할 로직들이 많이 존재한다면 반대방향 (양방향으로) 매핑도 해준다! 그리고 **양방향의 경우 연관관계의 주인은 외래 키의 위치를 기준으로 정하면 된다!!**

한번 더 간단히 정리하고 가자.

## 연관관계 매핑시 고려사항

### 다중성

- 다대일: @ManyToOne
- 일대다: @OneToMany
- 일대일: @OneToOne
- 다대다: @ManyToMany

이렇게 4가지 다중성이 존재해. 각각의 다중성 중 어떤 것을 선택하느냐는 DB ERD설계할 때와 마찬가지로 각각의 Entity 입장에서 관계를 정의하면 되는거야.

한 가지 예를들면, 회원과 이메일 Entity가 존재한다면 한 회원은 여러 Email을 가질 수 있고 여러 Email은 한 회원에 속할 수 있는 것이기에 회원과 이메일은 1:N 관계이고 @OneToMany (회원쪽 r기준에서)로 결정내릴 수가 있는거지.

### 단방향, 양방향

사실 단방향은 크게 문제될 것이 없어!! 그럼 뭐시 문제인가??

양방향 연관관계야!! why?? 이 차이를 좀 알아두고 가야해.

테이블은 외래 키 하나로 두 테이블이 연관관계를 맺어. 엄밀히 말하면 방향이란 것이 없이 외래키 하나로 두 테이블 모두 접근하는 거지.

그러나 객체 양방향 관계는 달라. A,B 객체가 양방향 연관관계를 맺고 있다면 엄밀히 말하면 A→B, B→A 이렇게 참조가 두 군데에 있다고 생각할 수 있어.

이 차이에서 오는 결론은 다음과 같애.

A→B, B→A 이렇게 두 참조가 있다면 **어떤 참조가 테이블의 외래키를 관리할 것인가??**

그리고 이를 결정해주는 것이 연관관계의 주인이라고 생각하면 되는거야!!

결과적으로

- 연관관계의 주인 : 외래키를 관리하는 참조
- 주인의 반대편 : 외래키에 영향을 주지 않으며 그렇기에 단순 조회만 가능해!

### 다중성과 방향을 설정할 때 고려할 것

다대일 단방향,양방향의 경우 설정하는 데 크게 무리가 없어!! 외래키를 관리하는 쪽이 연관관계의 주인이라 했는데 이 외래키를 관리하는 녀석은 테이블에서 외래키가 위치한 녀석과 매핑되는 엔티티를 하는 것이 좋아!!

그러나 일대다 단방향 관계를 본다면 단방향이기에 OneToMany의 One 쪽에서 외래키를 관리하는 구조가 되어버리는데 테이블에서는 Many쪽에 외래키가 존재해.

객체와 테이블의 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조가 되어버리고 이는 추천하지 않는 구조야!!

**→ 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하는 것을 권장해!!**

(일대다 양방향의 경우는 공식적으로 존재하지 않을 뿐더러 어떻게 구조를 구현할 수는 있겠으나 다대일 양방향을 사용하는 것이 좋아)

그럼 각각의 연관관계의 종류에 대해 조금 더 자세히 알아보자

## 연관관계의 종류

### 다대일(N:1) 단방향

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/09_modeling.png?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/09_modeling.png?raw=true)

**가장 많이 사용되고 있는 연관관계야.**

Member 쪽에서 Team 필드를 통해 Member에서 Team으로 참조하는 구조야.

(단방향이기에 Team에서 Member로 참조하는 것은 현재 구조에선 없지)

가장 객체적인 설계와 테이블적인 설계의 매핑이 잘 되는 구조라고 볼 수 있지.

그게 무슨 말이냐??

현재 테이블의 멤버와 팀의 관계는 N:1이고 그럼 당연히 외래키는 N쪽인 MEMBER TABLE에 존재하겠지. (이건 테이블 설계 상 무조건 이러한 구조를 가져야하는 것이야)

그리고 객체의 관계를 보았을 때에도 Member에서 Team 필드를 MEMBER TABLE의 외래키와 매핑을 시켜줌으로써 객체에서 외래키를 관리하는 놈(Member)와 실제 테이블 상에서 외래키가 존재하는 놈(MEMBER TABLE)이 매칭이 딱 된다는 말이지!!

```java
@Entity
public class Member {
	@Id @GenartedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	//연관관계 매핑해주는 부분!!
	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
```

위 처럼 코드상에서 셋팅해주면 돼!!

@ManyToOne 어노테이션을 붙여줌으로써 다대일 관계 매핑을 해줌과 동시에 @JoinColumn 어노테이션을 통해 어떤 외래키와 매핑 되는지의 정보를 주면 되는거야!!

### 다대일(N:1) 양방향

위의 다대일 단방향 관계를 참고해보면 Member에서 Team으로의 참조는 있었으나 Team에서 Member로의 참조는 없었따!

여기서 Team에서 Member로의 참조를 추가시켜주면 다대일 양방향 관계가 만들어지는 것이다.

아래 그림처럼 말이다!

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/11_many_to_one.PNG?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/11_many_to_one.PNG?raw=true)

이렇게 해줌으로써 이제 Team에서도 Member를 참조하는 것이 가능해 지는거야!!

저~ 위에서도 말했듯이 양방향은 엄밀히 얘기하면 단방향이 두개 존재한다는 것이라고 했는데 이제 그 말이 이해가 될거야. 결국은 반대쪽에서 단방향 관계를 추가시키니 양방향 관계가 만들어진거자나.

여기서 중요하게 보고 넘어가야할 것이 있어.

**현재 Member 쪽이 외래키를 관리하고 외래키를 관리하느 놈→ 연관관계 주인이기에**

**Team 쪽에서 어떤 DB에 영향을 주는 작업을 하지는 못해!!**

만들어준 해당 참조를 통해 조회만 가능하지 Update는 할 수 없다는 말이야!!

이를 코드로 나타내어 본다면

```java
public class Team {
    @Id @GenartedValue
		@Column(name = "TEAM_ID")
		private Long id;

		//연관관계 정의 해 주는 곳
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

}
```

이렇게 @OneToMany 어노테이션을 붙임으로써 연관관계를 정의 해 줄수 있으며 mappedBy를 통해 연관관계의 주인을 나타낼 수 있어!!

현재 Member 쪽이 외래키를 관리하고 있는 놈이며 그렇기에 연관관계의 주인이 되어야하고

→ Team 쪽에서 mappedBy를 해줌으로써 Member가 연관관계의 주인임을 정의할 수 있어.

### 일대다(1:N) 단방향

1이 연관관계 주인인거쥐. 먼저 그림으로 한번 스윽 보겠따.

[https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdMPMAv%2FbtqOn1xvo7I%2F7lfY8tXXcMiAfxV1ufN6n0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdMPMAv%2FbtqOn1xvo7I%2F7lfY8tXXcMiAfxV1ufN6n0%2Fimg.png)

Team이 1이고 Member가 N이었자나!!

위의 그림과 차이점을 보면 Team과 Member의 위치가 바껴있지.

여기서는 1 즉 Team 을 기준으로 외래키를 관리하는 등 해보겠따이거야.

그리고 단방향이기에 Team은 Member를 참조하고싶지만 Member에서는 Team을 참조하지 않아도 된다는 그런 설정이지!!

이렇게 되면 Team쪽에서 외래키를 관리하는데 TABLE 상에서 외래키는 당연히 N쪽인 MEMBER에 위치하고 있겠지.

결국엔 위처럼 조금은 이상한 구조가 만들어져 버리는거야.

이게 어떻게 돌아가는거냐??

보이는 그대로 Team에서 members에 뭔가 넣거나 혹은 수정이 있으면 MEMBER TABLE에 대해 이를 맞춰주는 작업들이 들어간다고 보면 되는거지.

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<Member>();

}
```

코드로 본다면 위처럼 생겼다고 볼 수 있찌!!

이렇게 구현할 경우 어떤 부분이 안좋을까?? 🧐🧐🧐

**쿼리가 많이 나가게 돼!!**

```java
public class JpaMain {
	public static void main(...) {
		... //em 만들고 transaction관련 처리 하고 try catch.. 등등
		...
		Member member = new Member();
		member.setUsername("member1");

		em.persist(member); //MEMBER TABLE INSERT쿼리 날라가

		Team team = new Team();
		team.setName("teamA");

		team.getMembers().add(member); // MEMBER TABLE 쪽에 UPDATE를 날려줘야해

		em.persist(team); //TEAM TABLE INSERT쿼리 날라가

		...
	}
}
```

외래키를 관리하는 객체 관점과 실제 테이블의 외래키가 존재하는 테이블의 매칭이 될 경우 위 경우처럼 insert를 하면서 update 쿼리를 날릴 필요가 없어!!

무슨 말이야?? team쪽의 getMembers().add를 통해 어떤 변화가 생겼는데 Team의 members가 관리하는 외래키는 MEMBER TABLE에 존재하고 결국 그 변화를 반영하기 위해서는 MEMBER TABLE에 UPDATE쿼리를 날릴 수 밖에 없는거지!!

그리고 실무에서 사용을 기피하는 이유는 **엄청 복잡한 상황에서 TEAM을 건드렸는데 MEMBER에 쿼리가 나가는 상황은 추적하기 어렵고 결과적으로 후에 관리하기 힘들어지는거야!!**

**→ 다대일 양방향 관계로 사용하는 것을 추천하는 이유야!!**

### 일대다(1:N) 양방향

공식적으로는 없지만 야매로 가능해..

[https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCRM2F%2FbtqOprboMgd%2F2mEJmR1vCSnvN50oPmxnY0%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCRM2F%2FbtqOprboMgd%2F2mEJmR1vCSnvN50oPmxnY0%2Fimg.png)

자... 지금까지 왔으니 해당 다이아그램이 코드상에서 어떻게 나타나 지는지 느낌은 올거야.

공식적으로 되는 것이 없고 야매로 한다는 것은 아래와 같애.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

		//양방향을 위한
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

}
```

Member 쪽에서 Team을 참조할 수 있는 구조로 만들었지.

@ManyToOne을 통해 Member에서 Team로의 관계를 만들어주었고

@JoinColumn 을 통해 외래키 매핑을 시켜줬지. 위에서 보듯이 옵션이 추가가 되었음을 알 수 있어. 만약 저 옵션이 없는 경우를 생각해보자.

그럼 연관관계의 주인이 Team, Member 둘 다가 되버리는 아주 이상한 상황이 만들어지는거지!!

그래서 insertable = false와 updatable = false 를 통해 조회만 가능하게 함으로써 야매로 해당 일대다 양방향 관계를 가능하게 끔 한거지

**→ 그냥 쓰지마!!!! 다대일 양방향을 써!!**

### 일대일 단/양 방향

일대일 관계는 외래키가 주 테이블에 있어도 되고 대상 테이블에 있어도 되는거야!!

why? 1:1이니까!! 그래서 두가지 방법 중 하나를 선택하면 되는거지.

그리고 데이터베이스에 유니크 제약조건을 걸어줌으로써 일대일 관계를 완성시킬 수 있어.

일대일은 결국 자기가 자기꺼 관리한다 생각하면 돼!!

테이블에 외래키가 있는 쪽과 매칭되는 놈을 연관관계 주인으로 설정하고 쓰는거쥐

1. **일대일 주 테이블에 외래 키 단방향**

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/13_one_to_one.png?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/13_one_to_one.png?raw=true)

회원이 하나의 락커를 가지고, 락커도 하나의 회원에게 할당되어진다는 상황을 가정하고 있어!!

그리고 주 테이블에 외래키가 있고 단방향이라면 위 그림처럼 되는거지!!

그냥 다대일 단방향 관계 매핑과 거의 동일한 구조로 만들면 돼!! (@OneToOne 어노테이션만 달라지는 부분이라고 볼 수 있쥐)

1. **일애일 주 테이블에 외래 키 양방향**

: 위 상황에서 Locker에서 Member를 참조할 수 있게끔 Locker 쪽에 추가해주면 되겠지.

간단하게 코드를 보자면

```java
@Entity
public class Locker {

    @OneToOne(mappedBy = "locker")
    private Member member;
}

```

이렇게 되는거겠지. 어렵진 않아.

1. **대상 테이블에 외래 키 단방향**

: 이건 그냥 안되는거야!!

1. **대상 테이블에 외래 키 양방향**

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/14_one_to_one.png?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/14_one_to_one.png?raw=true)

그냥 '일대일 주 테이블에 외래 키 양방향'을 뒤집은 구조라고 생각하면 돼!!

주 테이블(MEMBER)에 외래키가 있던게 대상 테이블(LOCKER)로 간 것만 다르고 전체적인 구조는 동일 한 것을 볼 수 있어.

### 다대다(N:M) 단/양 방향

: 이건 그냥 쓰지마. 실무에서는 사용이 안되어진다고 볼 수 있데.

WHY??

간단하게 이야기하자면 @ManyToMany를 통해 양방향 관계를 설정하면 결과적으로 관계를 나타내기 위한 중간 테이블이 만들어진데!!

하지만 이 중간테이블에 어떤 작업을 해줄 수 있는 방안은 없어. 단지 두 테이블의 관계만 각각의 외래키를 통해 형성해 줄 뿐이지.

그럼 만약 실무에서 형성된 관계 테이블에 시간(언제 형성되었는지)과 같은 정보를 넣고싶다고 하더라도 안되는거야!!

😅**😅😅그럼 어떻게 하란거야...😅😅😅**

**→ 연결 테이블용 엔티티를 추가하면 돼!! 즉 연결 테이블을 엔티티로 승격시키는거지**

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/18_many_to_many.png?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/18_many_to_many.png?raw=true)

이렇게 말이야!!

@ManyToMany 를 @OneToMany와 @ManyToOne을 통해 풀어내는거지!! 이렇게 사용하면 된다 이말이야!! 여기서 한가지 의문은 관계 테이블에서 보통 A,B의 관계를 나타내는 테이블이라고 가정한다면 A,B의 PK를 FK로 가져와 이 두개를 묶어 관계 테이블의 PK로 사용한다.

BUT 영한센세의 말에 의하면 PK는 최대한 비지니스 로직을 벗어나는 것이 좋고 의미 없는 것을 사용하는 것이 좋다고 하니 Generated Value를 통해 만들어서 사용하도록 하자!!

> [https://www.inflearn.com/course/ORM-JPA-Basic](https://www.inflearn.com/course/ORM-JPA-Basic)

> [https://ict-nroo.tistory.com/125?category=826875](https://ict-nroo.tistory.com/125?category=826875)

> [https://jgrammer.tistory.com/entry/JPA-다양한-연관관계-매핑-1-다대일-일대다-1](https://jgrammer.tistory.com/entry/JPA-%EB%8B%A4%EC%96%91%ED%95%9C-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91-1-%EB%8B%A4%EB%8C%80%EC%9D%BC-%EC%9D%BC%EB%8C%80%EB%8B%A4-1)
