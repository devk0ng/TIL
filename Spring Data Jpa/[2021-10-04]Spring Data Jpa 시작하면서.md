# 스프링부트 라이브러리 구조

### Spring-boot-starter-Web

- spring-boot-starter-tomcat
- spring-webmvc

<br/>

### spring-boot-starter-jdbc

- spring-boot-starter-aop
- spring-boot-starter-jdbc (HikariCp Connection Pool)
- spring-data-jpa

<br/><br/><br/>

# 스프링 부트를 통한 이점

: 부트를 이용하지 않을 경우 persistence.xml 부터 시작하여 프로그래머가 설정해 주어야하는 부분들이 많으나 부트를 사용할 경우 이런 부분을 자동으로 설정 해 준다.

<br/><br/><br/>

# JPA 쿼리 파라미터 로그 남기기

```bash
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.7'
```

해당 라이브러리 사용하면 돼!

<br/><br/><br/>

# 스프링 데이터 JPA에서 공통 인터페이스를 상속하면 어떻게 동작하는 것인가?

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

해당 코드를 보자.

extends JpaRepository<Member, Long> 부분이 있다.

해당 형태는 JpaRepository<엔티티, 엔티티의 key type> 으로 생각하면 된다.

이렇게 MeberRepository를 정의한 후 Service 계층에서 편히 주입받아서 사용하면 된다!!

이때 한가지 의문이 들것이다!! 현재 MemberRepository는 인터페이스고 구현체를 만들어주지 않았는데 어떻게 주입받아서 사용할 수 있는가??

이 질문의 답은 JPA가 해당 인터페이스를 본따서 프록시 객체를 만들어주고 이 프록시 객체를 주입받아 사용할 수 있게 되는 것이다!! (실제 클래스 타입을 찍어보면 proxy객체 확인가능)

<br/><br/><br/>

# 공통인터페이스 구성

![](https://backtony.github.io/assets/img/post/jpa/datajpa//1-1.PNG)

<br/><br/><br/>

# 주요 메서드

## 제네릭 타입

- T : 엔티티
- ID : 엔티티의 식별자 타입
- S : 엔티티와 그 자식 타입

<br/>

## 주요 메서드 종류

- save(S) : 새로운 엔티티 저장, 이미 있는 엔티티 병합
- delete(T) : 엔티티 하나를 삭제. (EntityManager.remove() 호출)
- findById(ID) : 엔티티 하나를 조회. (EntityManager.find() 호출)
- getOne(ID) : 엔티티를 프록시로 조회. (EntityManager.getReference() 호출)
- findAll(...) : 모든 엔티티를 조회. 정렬(Sort)나 페이징(Pageable) 조건을 파라미터로 제공할 수 있어

<br/><br/><br/><br/><br/><br/>

> [https://www.inflearn.com/course/스프링-데이터-JPA-실전](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%8D%B0%EC%9D%B4%ED%84%B0-JPA-%EC%8B%A4%EC%A0%84)
