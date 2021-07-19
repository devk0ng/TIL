# API Composition vs CQRS Pattern

마이크로서비스로 전체 구성을 하였다고 가정을 해보자!

특정 서비스를 제공하는 녀석을 A라고 가정해본닷. 그리고 A로 전달된 클라이언트의 request를 처리하기 위해서는 A친구는 다른 마이크로서비스 B,C,D에서 제공하는 정보를 조합해서 클라이언트에게 최종적인 결과를 주어야한닷

이때, 이 상황에서 사용할 수 있는 방법이 2가지 존재해!!

바로~ API Composition과 CQRS야!!

### 1. API Composition

먼저 api composition이라는 놈을 보자!! 

![https://i.imgur.com/NV9hOyi.png](https://i.imgur.com/NV9hOyi.png)

위 그림이 API Composition의 한 예야. 보다시비 sevice A,B,C가 존재하고 각각의 서비스들로부터의 결과들을 조합,조립 해주는 API composer가 있어!!

이렇게 각각의 결과들을 조립해주는 API composer를 통해 각각의 서비스의 데이터들을 모아 클라이언트에게 제공하는 방식을 API Composition이라고 해!!

이 친구의 장점을 뭘까?? 바로 Simple and Simple and Simple! 이야!! 만약 **조합해야하는 데이터가 많지 않다면** 이 방법을 고려해볼 수 있지!! 

B.U.T 여러 서비스들을 호출해서 조합해야하는 경우 위 방법은 좋지 않을 수 있어. 

왜냐하면 결과적으로 API Composer가 메모리상에서 각각의 결과들을 조합해야하고 이는 규모가 커지면 부담으로 다가와!! 그리고 각각의 서비스를 호출하는 부분에 있어서 만약 동기방식으로 만들어졌다면 Delay가 발생할수도 있을거구 각각의 서비스들로 보낸 요청이 잘왔는지 혹은 따로 검사해봐야할 조건들을 처리하는 로직들이 추가되어질텐데 이러한 부분도 규모가 커지면 우리 composer 친구가 부담이 되어지지!!

그래서 그 다음 방법을 생각해낸거쥐 😮

### 2. CQRS

![https://i.imgur.com/ZI1mODa.png](https://i.imgur.com/ZI1mODa.png)

CQRS에 대해서는 따로 정리를 하기도 했었으니 간단하게 스슥하고 넘어가겠습니다!!

CUD 즉 시스템의 정보를 변경하는 작업들이 발생하면 해당 정보를 포함한 Event를 발행시키고 Query를 담당하는 쪽에서 해당 Event를 구독하고 있기에 이를 받게 될 때 변경된 데이터에 대한 동기화 작업을 Query용 Database에 하는거야!! 

이렇게 될 경우 그 전 API Composition에서 발생하는 문제점들을 해결할 수 있어!!

B.U.T 위 그림에서도 느낌이 오지만 확실히 API Composition방법에 비해서는 더 Complex하다는 단점이 존재해.

그때그때 상황에 맞추어 더 효율적인 방안을 택하면 될 거 같다 이말이야!! 

> 출처 : [https://learn.co/lessons/microservices-patterns-chapter-7](https://learn.co/lessons/microservices-patterns-chapter-7)