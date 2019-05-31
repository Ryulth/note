## Event Sourcing Pattern for Concurrency

 요즘 들어 동시성 처리를 중요시하는 어플리케이션을 제작하게 되었다. (공동 편집 기능, 경매 프로그램 등등) 그런데 동시성 동기화 라는 키워드는 OS 에서 들어보았을 것이다. 크리티컬 섹션을 만들어서 동시에 접근을 못하게 막고 순차적으로 처리하여라, Race Condition 등등 사용하고 데드락이 발생하지 않도록 하여라 하면서 많이 배웠을 것이다. 

나는 이벤트 소싱 패턴으로 동시성 문제를 해결 하였다. [이벤트 소싱](<https://github.com/Ryulth/note/blob/master/Event-Sourcing-Pattern.md>)

### 기존의 방식

트랜잭션 처리 혹은 크리티컬 섹션을 만들어 동시성 처리를 하였다. 

예를 들어 지금 잔액 1000원

A 라는 사람이 2000원을 입금하고 B 라는 사람이 같은 계좌에 3000원을 입금한다고 하자. 그러면 트랜잭션 처리를 통해 A 라는 사람의 요청먼저 처리를 하고 B 라는 사람의 요청을 처리 한다고 하면 B라는 사람은 1000원인 상태에서 입금을 하였는데 1000+2000+3000 인 6000원의 결과를 받게 된다. 입금이니까 뭐 그러려니 하겠지만 더욱 중요한 데이터라면 요청을 reject 처리 해야 할 수 도 있다. 사실  두개 이상의 요청이 동시에 들어왔을 때 를 catch 할 수 만있다면 동시요청에 대한 비즈니스 로직을 구상할 수 있다. 하지만 기존의 방식에서는 락을 걸어 대기 상태로 놓고 실행 시켜주기 때문에 데드락과 같은 상황이 발생 할 수 있다.

### 이벤트 소싱 방식

이벤트 소싱 방식을 사용하면 버젼관리를 할 수 있다. 깃 같은 느낌이라 하면 이해하기 쉬울 수 있다. 깃은 사실상 현재상태를 가지고 있고 그 이전 상태에 대한 정보는 diff 정보만 가지고 있다. 그래서 머지 할때 base 버젼이 같은지 보고 알아서 잘 합쳐주는 편이다. 이런 느낌이다.

이벤트 소싱도 각 이벤트에 버젼값을 부여해서 버젼으로 동시성 체크를 할 수 있다.

아까와 같은 상황을 이벤트 로그로 보면

| 주체   | eventType | metaData | version |
| ------ | --------- | -------- | ------- |
| 관리자 | setting   | 1000원   | 1       |
| A      | 입금      | 2000원   | 2       |
| B      | 입금      | 3000원   | 2       |

이런식으로 들어온다고 하면 버젼 충돌이 일어난다. 따라서 둘 중 한 이벤트는 정책에 따라 버젼을 올려서 넣던지 reject 하던지 해야한다.

버젼 값이 충돌하면 동시에 들어왔다고 볼 수 있으므로 이때 동시성 처리 비즈니스 로직을 넣어주면 된다.

버젼값만 잘 관리해주면 이벤트 스트림을 가지고 동시성을 유지할 수 있고 불변성을 유지 할 수 있다.

### Redis Stream 사용

버젼관리에 가장 간단하게 그리고 잘 어울리는 플랫폼을 찾았다. 

Redis 5.0 ++ 에 있는 stream 형태인데 새로 넣는 데이터 id 값이 이전데이터의 id 값보다 커야한다. 

버젼관리에 용이 할 것이다.  좋아보여서 적용해보았다. 

이벤트 소싱패턴에 가장 최적한 형태로 보인다.  [더보기](<https://github.com/Ryulth/note/blob/master/Redis-Streams-with-RedisTemplate.md>)

### TODO

* CQRS 패턴 이해



#### reference

- [SPRING CAMP 2017 천보님 발표자료](https://github.com/jaceshim/springcamp2017/blob/master/springcamp2017_implementing_es_cqrs.pdf)








