# queryDSL 기초(조인, 서브쿼리, case, 상수)

# 조인

## 기본 조인

<br/>

### 사용방법

join( 조인할 대상, 별칭으로 사용할 Q Type)

<br/>

### 예시

```java
//member와 team의 QType static import한 상태
List<Member> result = queryFactory
              .selectFrom(member)
              .join(member.team, team)
              .where(team.name.eq("teamA"))
              .fetch();
```

- join(), innerJoin() : inner join
- leftJoin() : left outer join
- rightJoin() : right outer join

<br/><br/>

## 세타 조인 (연관관계가 없는 조인)

<br/>

### 사용방법

from() 절에 조인하고자 하는 대상 (연관관계가 없는) 엔티티를 선택하여 넣어주면 돼!

<br/>

### 예시

```java
List<Member> result = queryFactory
              .select(member)
              .from(member, team)
              .where(member.username.eq(team.name))
              .fetch();
//cartesian product 형태의 결과에서 조건에 해당하는 것을 걸러준 결과를 뱉어내줘
```

외부조인(위의 일반조인 설명에서 제공되어지는 left outer or right outer)은 안돼!!

but, 조인 on 을 이용하면 똑같은 효과를 볼 수 있어. 즉 외부조인 형태가 가능해져.

<br/><br/>

## 조인 + on 절

<br/>

### 사용방법

join().on() 형태로 체이닝 걸어서 사용하면 돼

<br/>

### 조인 대상 필터링 할때 사용

```java
List<Tuple> result = queryFactory
              .select(member, team)
              .from(member)
              .leftJoin(member.team, team).on(team.name.eq("teamA"))
              .fetch();
/*
회원과 팀을 조인하면서 팀 이름이 A인 팀으로 조인 대상에 대해 필터링해서 가지고와. 물론 회원은 모두 조회
sql : SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and
  t.name='teamA
*/
```

참고로 내부조인을 사용하면서 대상에 대해 필터링을 하기위해 on절에 조건을 거는 것은 where 절에 조건을 거는 것과 동일해. 

즉 내부 조인이면 익숙하게 where 절에 필터링 조건을 넣어주는 것이 편하겠지.

<br/>

### 연관관계가 없는 엔티티의 외부 조인을 원할 때 사용

```java
//회원의 이름과 팀의 이름이 같은 대상 외부 조인 원하는 경우
List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(team).on(member.username.eq(team.name))
                .fetch();
//일반조인은 leftJoin(member.team, team) 이렇게 조인하고자하는 대상, Q타입이 들어가지만
//연관관계가 없는 엔티티의 외부조인을 원활 때는 위 예시처럼 leftJoin(team).on()이야
```

- 하이버네이트 1.5버전부터 on 절을 이용하여 서로 관계가 없는 필드로 외부 조인하는게 가능해졌데.
- 위에서 보듯이 일반 Join과는 조금 문법이 달라.

<br/><br/>

## 페치조인

<br/>

### 사용방법

join().fetchJoin() 이런 식으로 체이닝 해주면 돼!!

<br/>

### 예시

```java
Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team, team).fetchJoin()
                .where(member.username.eq("member1"))
                .fetchOne();
```

- 패치 조인은 잘 알겠찌만 sql에서 제공하는 기능은 아니고, sql join을 이용해서 연관된 엔티티를 한꺼번에 가져오고 싶을 때 사용하는 기능이야.

<br/><br/><br/>

# 서브쿼리

<br/>

## 사용방법

com.querydsl.jpa.JPAExpressions를 사용해서 원하는 서브쿼리를 작성하면 돼!

<br/>

## 예시

```java
//where 절에서 서브쿼리 사용하기
List<Member> result = queryFactory
              .selectFrom(member)
              .where(member.age.eq(
                      JPAExpressions //이런식으로 말이야!!
                              .select(memberSub.age.max())
                              .from(memberSub)
)) .fetch();

//select 절에서 서브쿼리 사용하기
List<Tuple> fetch = queryFactory
          .select(member.username,
                  JPAExpressions
                          .select(memberSub.age.avg())
                          .from(memberSub)
          ).from(member)
          .fetch();

//JPAExpressions를 static import 해주면 조금 더 이쁘게 작성할 수 있어
```

위처럼 select 절과 where 절에서 서브쿼리를 사용할 수 있지만 from절에서는 사용하지 못해!!

why..? jpa의 jpql에서 지원하지 않아!! 결국 queryDsl은 jpql로 변환이 될거고 jpql에서 지원하지 않는 기능은 지원하지 못해!!

그럼 어떻게 하느냐?? from 절에 서브쿼리가 필요한 경우는 아래로 대체

1. 서브쿼리를 join으로 변경 시도 (가능한 상황도 있지만 불가능할수도..)
2. 어플리케이션의 쿼리를 2번 분리해서 실행
3. nativeSQL을 사용

<br/><br/><br/>

# Case 문
<br/>

## 단순한 경우

<br/>

### 예시

```java
List<String> result = queryFactory
						.select(member.age 
								.when(10).then("열살")
								.when(20).then("스무살")
								.otherwise("기타")) .from(member)
						.fetch();
//이런식으로 select의 대상에 .when().then() 체이닝해서 사용하면 돼
```

참고로 DB에서는 raw데이터를 그루핑 하거나 필터링 할 순 있겠지만 최소화 시켜야해!!
디비에서 이러한 부분들을 맡는 것은 좋지 않데.
즉, 위 경우면 가져온 데이터를 바탕으로 어플리케이션에서 10이면 열살, 20이면 스무살 이런식으로 하는게 좋다는거지.

<br/><br/>

## 복잡한 경우

복잡한 경우에는 CaseBuilder() 라는 놈을 따로 사용해주면 돼!!

```java
List<String> result = queryFactory
             .select(new CaseBuilder()
									.when(member.age.between(0, 20)).then("0~20살") 
									.when(member.age.between(21, 30)).then("21~30살") 
									.otherwise("기타"))
             .from(member)
             .fetch();
//요렇게 써주면 돼
```

<br/>

만약에 특정 case에 대해 순서를 부여하고 싶다면??

예를들어서 아래와 같은 조건의 경우

1. 0~30 살이 아닌 회원 가장 먼저 출력
2. 0~20 살 회원 출력
3. 21~30 살 회원 출력

<br/>

```java
NumberExpression<Integer> rankPath = new CaseBuilder()
             .when(member.age.between(0, 20)).then(2)
             .when(member.age.between(21, 30)).then(1)
             .otherwise(3);

List<Tuple> result = queryFactory
	           .select(member.username, member.age, rankPath)
						 .from(member)
             .orderBy(rankPath.desc())
             .fetch();

//result를 출력해보면 case문에서 정의한 랭크의 내림차순으로 정보를 출력함을 볼 수 있어
```

<br/><br/><br/>

# 상수, 문자 더하기

<br/>

## 상수

<br/>

### 사용방법

queryDsl에서 제공하는 Expressions.constant(xx) 사용하면 돼!!

<br/>

### 예시

```java
Tuple result = queryFactory
            .select(member.username, Expressions.constant("A"))
            .from(member)
            .fetchFirst();
//위 처럼 굳이 sql에 저 상수값을 넘겨주지 않아도 결과를 구할 수 있을 때에는
//알아서 sql에 constant값을 넘기지 않게 끔 최적화가 돼
```

<br/><br/>

## 문자 더하기

<br/>

### 사용방법

.concat() 을 체이닝해주면 돼!

<br/>

### 예시

```java
String result = queryFactory
            .select(member.username.concat("_").concat(member.age.stringValue()))
            .from(member)
            .where(member.username.eq("member1"))
            .fetchOne();
```

위 예시에서 member.age.stringValue()를 보자.

java는 타입이 정해져있는 언어이기에 저런식으로 string으로 변환을 해주어야해.

stringValue()를 통해 문자가 아닌 타입을 문자로 변환할 수 있는데 자주 쓰인다니 기억!! (Enum을 처리할 때)

<br/><br/><br/><br/><br/><br/><br/>

> [https://www.inflearn.com/course/Querydsl-실전](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84)
>