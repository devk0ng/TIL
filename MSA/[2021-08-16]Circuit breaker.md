# Circuit breaker

## Circuit Breaker는 어디에 쓰이는 놈인가?

**circuit breaker**가 무엇을 하는 놈인가를 보기 전에 어디에 쓰는지를 먼저 보자

**MSA구조에서는 시스템을 여러개의 서비스 컴포넌트로 나누고 이러한 서비스 컴포넌트간에 호출하는 개념**을 가지고 있지.

이러한 구조에서 잘 생각해보면 단점이 몇 가지 존재하는 데 그 중 하나가 **장애전파가 일어날 수 있다는 점**이야!

장애 전파가 무엇이냐??

우선 그림을 하나 들구 올게

![](https://t1.daumcdn.net/cfile/tistory/99E6754C5AC39FAA08)

이런 구조를 한번 생각해보자.

Service A가 Service B를 호출하는 상황에서 어떤 문제로 인해 Service B가 응답을 못하거나 아니면 그 응답의 속도가 매우 느리다고 생각해보자.

그럼 Service A에서는 어떤 일이 발생하게 될까?? 당연히 Service B의 응답을 각각의 스레드(실행단위)들이 기다리고 있는 상황이 되겠지??

그 상황에서 Client의 요청으로 Service A를 계속해서 호출해야하는 상황이라면?? 아마 전체 시스템 장애로 번지겠지...??

이러한 것을 장애전파라고 해.

그리고 이러한 문제를 해결하는 디자인 패턴이 Circuit Breaker Pattern 이야!!

**핵심은 서비스간 호출에서 그 사이에 Circuit Breaker라는 중재하는 놈을 설치한다고 생각하면 될 것** 같애.

![](https://t1.daumcdn.net/cfile/tistory/99427C475AC39FAA08)

위 그림처럼 말야!!

그리하여 Service B로의 호출은 Circuit Breaker를 통해서 할 수 있게 구조를 함으로써 B가 정상적인 상황에서는 Circuit Breaker가 B 서비스를 호출해주고 만약 Service B에 문제가 발생하였음을 감지하게 된다면 Service B로의 호출을 강제적으로 끊으면서 Service A의 스레드들이 더이상 기다리지 않아도 되게 해주는 것이다!!

<br/><br/>

## Fall-back messaging

위처럼 구조를 짤 경우 장애전파는 막을 수 있으나 Service A에서 이에 대한 장애 처리 로직이 별도로 필요하게 되겠지.

이게 너무 귀찮으니까 조금 더 발전시킨 게 바로~ Fall-back 메시징이란 것이라고 하네!!

장애를 감지하였을 때 정의된 룰에 따라 Circuit Breaker가 그때그때 상황에 맞는 메시지를 리턴하게하는 방법이야!!

![](https://t1.daumcdn.net/cfile/tistory/993FD73B5AC39FAA17)

위의 그림처럼 말이지!!

한가지 예를 들자면, Service A가 상품목록을 화면에 뿌려주는 놈이고 Service B는 머신러닝 기술을 도입해 상품 추천 목록 데이터를 뱉어내는 놈이라고 가정해보자. A는 B로부터 데이터를 받아 그 데이터를 바탕으로 화면에 뿌려주는 것이 전체적인 흐름이야!

이러한 상황속에서 B에 장애가 발생하였을 때 Circuit Breaker에서 개인별 추천이 아닌 정의되어있던 상품 목록들을 메시지로 뱉어냄으로써 장애가 발생하였으나 그 서비스가 죽지않고 이용할 수 있게 해주는거야!!

(물론 개인별 추천은 안되기에 사용자의 만족도는 감소할 수 있따)

Netflix에서 MSA구조로 해당 서비스를 만들었는데 이때 Circuit Breaker개념을 도입하였고 그렇게 개발한 것이 Hystrix라는 놈이다.

그리고 이를 오픈소스로 내놓았고 Spring에서 이를 보고 사용하기 편리하게 해주어서 Spring에서 쉽게 Hystrix를 통해 Circuit Breaker 패턴을 적용할 수 있다!!

나중에 직접 해볼 것!!

이상!!

<br/><br/><br/><br/><br/><br/><br/><br/>

> [https://bcho.tistory.com/1247?category=431297](https://bcho.tistory.com/1247?category=431297)
