# JPA 영속성 전이(CASECADE)와 고아객체

# 영속성 전이 (CASECADE)

영속성 전이란 특정 **엔티리를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때** 사용하면 되는 개념이야

예를 들어 부모 엔티티를 persist할 때 자동으로 자식 엔티티도 함께 persist 시키고 싶을 때 말이지!

```java
@Entity
public class Parent {
	,,,
	@OneToMany(mappedBy = "parent", casecade = CascadeType.ALL)
	private List<Child> childList = new ArrayList<>();
	..
}
```

이런 식으로 되어있다고 생각해보자. Parent와 Child 두 엔티티가 있고 위와 같은 관계를 가지고 있어. 이때 여기서 Parent의 멤버 중 child를 나타내는 친구에게 casecade all 옵션을 주면 어떻게 되냐!?

![](https://media.vlpt.us/images/syleemk/post/f4f0a051-3f01-4b1c-b7b2-14ffa478319f/image.png)

이렇게 되는거야!!

parent를 persist하게 될 경우 해당 parent 객체의 childList에 있는 child 정보에 대해서도 persist를 해주는거지!! 이게 영속성 전이야!!

참고로 영속성 전이는 연관관계를 매핑하는 것과는 아무런 관련이 없어!! 단지 **엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함**을 주는거지

위의 상황을 빗대어 설명하자면 CHILD의 소유자 PARENT가 하나일 때!! 다른 곳에서도 CHILD를 소유한다? 그러면 이러한 옵션은 사용하지 않는 것이 좋아!!

<br/>

## CASECADE의 종류

여러가지 있으나 아래 3가지, 그 중에서도 상위 2개만 알아두면 된데. 나머지는 잘 안쓰나봐

- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제

<br/><br/>

# 고아 객체

고아 객체 제거를 어노테이션의 속성을 통해 옵션을 줄 수 있어!!

고아 객체 제거가 무엇이냐??

**부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제** 해 주는거야

연관관계를 나타내는 어노테이션의 속성에 orphanRemovel = true 를 줌으로써 설정할 수 있어.

```java
Parent parent1 = em.find(Parent.class, id);
parent1.getChildren().remove(0);
```

그렇게 될 경우 위 코드에서 만약 parent 쪽에서 orphanRemovel 설정을 했다고 가정한다면

remove(0)에 해당하는 코드가 실행되면 database에서 해당하는 child도 지워주게 되는거지

<br/>

## 고아 객체 주의사항

- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는거야!
- 즉, 참조하는 곳이 하나일 때 사용해야하는 거지!!
- 그 관계를 특정 엔티티가 개인 소유할 때 라고 생각하면 좋을 거 같고 이때 사용할 수 있는거야.
- @OneToOne, @OneToMany일 때만 가능해

<br/><br/>

# 영속성 전이 + 고아객체

CascadeType.ALL + orphanRemovel=true 두 설정을 하게 될 경우 어떻게 될까??

**부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있게 되는거야**

DDD 에서 Aggregate Root 개념을 구현할 때 유용하다고 하는데 이 부분은 이후에 경험하게 될 때 자세히 찾아보자!!

<br/><br/>
<br/><br/>
<br/><br/>

> [https://www.inflearn.com/course/ORM-JPA-Basic](https://www.inflearn.com/course/ORM-JPA-Basic)
