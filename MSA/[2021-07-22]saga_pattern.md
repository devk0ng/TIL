# Saga Pattern

우선 먼저 MSA구조 이전에는 분산 데이터베이스에 대해 어떻게 트랜잭션을 만족시켰는지 보자!! 

이때는 **2-Phase Commit**이라는 방법을 사용했어!!

참고로 사용하는 데이터베이스가 분산 트랜젝션을 지원해야 사용할 수 있는 방법이야!! (같은 종류의 데이터베이스이어야 하기에 MSA구조에는 적합하지 않겠지)

![https://user-images.githubusercontent.com/14002238/113822026-dd260d80-97b7-11eb-8ea8-b3c939d08cec.png](https://user-images.githubusercontent.com/14002238/113822026-dd260d80-97b7-11eb-8ea8-b3c939d08cec.png)

큰 흐름은 위와 같애.

2-Phase Commit에서는 일반적으로 단일 노드 트랜잭션에서는 보이지 않는 Coordinator라는 놈을 사용해.

이 친구가 뭐하는 놈이냐면 각 데이터베이스의 commit할 수 있는 상태의 여부를 체크 해 주는 놈으로 중간에서 관리해 주는 녀석이라고 이해하면 쉬울 거 같애.

큰 순서는 아래와 같아.

1. 그 시작은 먼저 어플리케이션이 정상적으로 여러 데이터베이스 노드에서 데이터를 읽고쓰는 작업을 하는 것이야.
2. 그 이후에 커밋할 준비가 완료되면 Coordinator(트랜잭션 관리자)는 1단계를 시작해!! 1단계는 각 노드에 준비 요청을 보내어 커밋 가능 여부를 묻게 되며, 그 응답에 따라 커밋할지 롤백할지 여부를 결정해
3. 모든 노드가 commit이 가능하다는 응답을 했을 경우 Coordinaor는 2단계에서 커밋 요청을 전송하고 커밋이 수행되는거야
4. 만약 가능하지 않다는 응답이 온 경우 연관된 모든 노드에 중단 요청을 보내!

하지만 **MSA에서는 위와 같은 방법을 사용할 순 없어** 😂😂😂

MSA는 Service 별 Database를 가지고 있는 구조이며 각 Service는 물리적으로 분리되어 돌아가는 시스템이라고 볼 수 있지. 그렇기에 간단하게 local ACID transaction을 사용하는 것은 impossible이야! NoSQL의 경우는 2PC가 support되지도 않아

그럼 어떻게 해야할까?? 🤔🤔🤔

**→ 그래서 등장한 것이 SAGA Pattern 이야!!**

what is SAGA....

![https://chrisrichardson.net/i/sagas/From_2PC_To_Saga.png](https://chrisrichardson.net/i/sagas/From_2PC_To_Saga.png)

SAGA는 sequence of local transactions 이라고 할 수 있어. 수행해야할 일련의 local transaction들이 있을 거자나? 각각의 local transaction은 데이터베이스를 update한 후에 message나 event를 발행해!! 

왜?? 그 message나 event는 다음 local transaction을 발생시키는 장치가 되는거지!!

만약 특정 local transaction이 fail 했다면 SAGA는 보상 트랜잭션을 실행해!! 

보상 트랜잭션은 특정 local transaction이 fail했다고 가정할 때 그 특정시점 이전에 진행되었던 local transaction에서 db에 update하는 자기들만의 local transcation을 수행 하였을텐데 이를 원 상태로 돌려주는 작업이라고 생각하면 될 것 같애!!

아래 그림을 보면 조금 더 와 닿을꺼야.

![https://www.baeldung.com/wp-content/uploads/sites/4/2021/04/Figure-3-1024x636.png](https://www.baeldung.com/wp-content/uploads/sites/4/2021/04/Figure-3-1024x636.png)

이렇게 모든 operation은 compensating transaction 즉 보상 트랜젝션에 의해 롤백되어질 수 있어!!

SAGA 패턴에는 두가지가 존재해.  Choreography 방식과 Orchestration 방식이 있어.

### 1. SAGA Choreography

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fw5pQl%2FbtqBtFtm7ax%2FEtSFfrFCa9cKsXyrZiIk3k%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fw5pQl%2FbtqBtFtm7ax%2FEtSFfrFCa9cKsXyrZiIk3k%2Fimg.png)

Choregoraphy방식은 각각의 서비스가 자신의 local transaction을 관리하고 트랜잭션이 종료되면 완료 event를 발행합니닷!! 그럼 그다음 실행되어야 할 로컬 트랜젝션을 관리하는 App에서는 그 event를 수신받고 다음 작업을 처리하는거지!!

이때, Event는 Kafka같은 MQ를 통해 Event Drivent Architecture 형태로 보통 구성해!!

그럼 만약 실패한다면...?? 

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbigQxE%2FbtqBujXdUwD%2FULK7mKyfwdmDujlPJj3gcK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbigQxE%2FbtqBujXdUwD%2FULK7mKyfwdmDujlPJj3gcK%2Fimg.png)

위에서 말했듯이 Choregoraphy방식은 각 App별로 트랜잭션을 관리하는 로직이 있어! 따라서 중간에 만약 transaction을 실패하면 실패한 APP에서 실패한 정보에 대한 event를 발행하고 이를 수신한 쪽에서 보상 로직을 수행하며 Rollback 처리를 하는거야!! 

### 2. SAGA Orchestartion

Orchestartion은 트랜잭션 처리를 위한 SAGA 인스턴스(Manager)가 별도로 존재해!! 

트랜젝션에 관여하는 모든 서비스는 Manager에 의해서 점진적으로 트랜잭션을 수행하며 결과를 Manger에게 전달해줘. 

그렇게 진행하다 마지막 트랜잭션이 끝나게되면 Manager를 종료하면서 전체 트랜잭션 처리를 종료해.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb97dmZ%2FbtqBs0EBjbO%2FEekNphZUWmwKQhza29KJp1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb97dmZ%2FbtqBs0EBjbO%2FEekNphZUWmwKQhza29KJp1%2Fimg.png)

이러한 형식으로 말야!! 

그런데 만약 실패하게 된다면??

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBHcqB%2FbtqBr8iO9TH%2FNSZPa0sxoVEj57FZt4nBo1%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBHcqB%2FbtqBr8iO9TH%2FNSZPa0sxoVEj57FZt4nBo1%2Fimg.png)

실패한다면 Manager에서 보상 트랜잭션을 발동하여 일관성을 유지할 수 있게 해!! 이렇게 모든 관리를 해주는 Manager라는 놈이 존재하니 중앙에서 컨트롤 하는 놈을 집중적으로 신경쓰면 되기에 복잡성이 줄어들고 구현과 테스트가 상대적으로 쉬워!! 

하지만 이를 관리하는 Manager 즉 Orchestrator 서비스가 추가되어야 한다는 단점이 있어.

이상!

> [https://microservices.io/patterns/data/saga.html](https://microservices.io/patterns/data/saga.html)

> [https://dongwooklee96.github.io/post/2021/03/26/two-phase-commit-이란/](https://dongwooklee96.github.io/post/2021/03/26/two-phase-commit-%EC%9D%B4%EB%9E%80/)

> [https://cla9.tistory.com/22](https://cla9.tistory.com/22)
