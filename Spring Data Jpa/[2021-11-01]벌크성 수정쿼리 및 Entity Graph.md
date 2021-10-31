# 벌크성 수정쿼리 및 Entity Graph

# 벌크성 수정쿼리

: 일반적으로 데이터의 update는 jpa의 dirty checking 기능을 활용한다! 그러나 여러 데이터들에 대해 일관된 업데이트 작업을 진행할 때에는 한번의 쿼리로 update 반영하는 것이 좋으며 이때, 벌크성 수정쿼리를 사용하면 되는 것이다!!

<br/>

## JPA 벌크성 수정쿼리

: Spring Data Jpa에서 어떻게 벌크성 수정쿼리를 제공하는지에 대해 보기전에 Jpa만으로 어떻게 사용할 수 있는지를 보자

```java
public int bulkAgePlus(int age) {
 int resultCount = em.createQuery(
 "update Member m set m.age = m.age + 1" +
 "where m.age >= :age")
 .setParameter("age", age)
 .executeUpdate();
 return resultCount;
}
```

이런식으로 엔티티 메니저의 createQuery 메소드에 쿼리를 작성해 줌으로써 사용할 수 있다.

<br/>

## Spring Data Jpa 벌크성 수정쿼리

: 그럼 Spring Data Jpa를 사용할 경우 얼마나 간편해지는지를 한번 보자.

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

이렇게 간단하게 사용할 수 있다!!

벌크성 수정 쿼리는 기존 레파지토리 쿼리 메소드에 @Query 어노테이션에 직접 쿼리를 작성하던 방법과 동일하게 사용하면 되나 @Modifying 어노테이션을 붙여주어야 한다!!

그렇지 않을 경우 에러(org.hibernate.hql.internal.QueryExecutionRequestException: Not supported for DML operations)를 만날 수 있다..

<br/>

## 벌크성 수정쿼리 사용 시 주의할 점

: @Modifying 어노테이션의 속성으로 (clearAutomatically = true) 해당 값을 셋팅하지 않으면 문제가 발생할 수 있다.

어떤 문제냐??

벌크 연산은 영속성 컨텍스트를 무시하고 실행해. 그렇기 때문에 영속성 컨텍스트에 있는 엔티티의 상태와 DB의 엔티티 상태가 달라질 수 있어

그럼 어떻게 해야하냐? 아래 둘 중 하나의 방법을 택하면 되겠지

1. 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산 먼저 실행
2. 부득이하게 영속성 컨텍스트에 엔티티가 있으면 벌크연산 직후 영속성 컨텍스트를 초기화 해줘!! → @Modifying(clearAutomatically = true)를 통해 가능해!!

<br/><br/>

# EntityGraph

<br/>

## @EntityGraph란

: 엔티티 그래프는 연관된 엔티티들을 SQL 한번에 조회하는 방법이야. 보통 일반적으로 연관관계에 있는 데이터의 로딩은 지연로딩을 사용해!

이때, 연관된 엔티티들을 한번에 조회하려면 페치 조인을 사용하지.

EntityGraph를 사용하면 jpql 없이 페치조인을 사용할 수 있어! 물론 jpql + entity graph도 가능해

```java
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

//JPQL + 엔티티 그래프
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//메서드 이름으로 쿼리에서 특히 편리하다.
@EntityGraph(attributePaths = {"team"})
List<Member> findByUsername(String username)
```

이렇게 사용할 수 있어

페치 조인을 조금 더 간편하게 쓸 수 있다 정도의 느낌이라 생각하면 될 것 같애. Left Outer Join 사용해

<br/>

## NamedEntityGraph

: 네임드 엔티티 그래프라는 것도 있어!!

```java
@NamedEntityGraph(name = "Member.all", attributeNodes =
@NamedAttributeNode("team"))
@Entity
public class Member { .... }

//이렇게 JPA Named Query처럼 Entity에 @NamedEntityGraph 설정해주고

@EntityGraph("Member.all")
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

//요렇게 사용하면 되는거야
```

<br/><br/><br/><br/><br/><br/><br/>

> [https://www.inflearn.com/course/스프링-데이터-JPA-실전](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84)
