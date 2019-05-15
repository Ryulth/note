# Redis Streams with RedisTemplate

Redis 5.0 부터 지원된 자료형태 [Redis reference documentation](https://redis.io/topics/streams-intro).

### Redis Stream 특징

* 메시징 큐로 Kafka 를 많이 사용하는데 Redis 도 메시징 큐 형태로 사용할 수 있도록 지원한다.
* 형태는 기존의 key 이외에 ID 값이 존재한다. 이 ID 값은 자동으로 설정 가능하지만 기본적으로 **기존의 ID 보다 커야한다 **. 
* ID 값을 1-0 다음에 1-0 의 ID 를 부여하면 에러가난다.
* 1-10이 있는데 1-3 를 넣어도 에러가남.
* 이 기능으로 **이벤트 소싱 패턴**에서의 버전 충돌에 의한 동시성활용을 더욱 쉽게 구현 가능할 것으로 보인다.

### 사용법

* 일단 Redis 5.0 + 는 window 지원이 안됨 (좌절했으나 윈도우에 우분투 지원) -> [윈도우 10 WSL 사용](<https://docs.microsoft.com/ko-kr/windows/wsl/install-win10>)
* Jedis 와 Lettuce  가 있는데 spring-data-redis 에서 아직 stream 형태는 Lettuce 만 지원해준다.

> Redis Stream support is currently only available through the Lettuce client as it is not yet supported by Jedis.

Spring에서는 주로 RedisTemplate 를 사용한다. [spring-data-redis](<https://github.com/spring-projects/spring-data-redis>) 패키지를 사용한다. 2.2.0 버전부터 stream이 지원된다. 하지만 아직 2.1.7.RELEASE 가 최신 Release 버전이기 때문에 BUILD-SNAPSHOT 버전을 사용해야한다.

build.grade

```
repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-milestone" }
    maven { url "https://repo.spring.io/libs-snapshot" }
}

dependencies {
	*
	*
	*
    compile group: 'org.springframework.data.build', name: 'spring-data-parent', version: '2.2.0.BUILD-SNAPSHOT', ext: 'pom'

    compile "org.springframework.data:spring-data-commons:2.2.0.BUILD-SNAPSHOT"
    compile "org.springframework.data:spring-data-keyvalue:2.2.0.BUILD-SNAPSHOT"
    compile "org.springframework.data:spring-data-redis:2.2.0.BUILD-SNAPSHOT"

    compile group: 'io.lettuce', name: 'lettuce-core', version: '5.1.6.RELEASE'
	
	*
	*
	*
}
```

dependency를 이렇게 추가해주면 된다. (살짝 애먹은 부분, 공식 doc 에서는 compile "org.springframework.data:spring-data-redis:2.2.0.BUILD-SNAPSHOT" 이 부분만 언급되어 있음 )

* 나중에 2.2.0.RELEASE 버전이 나온다면 변경이 필요 할 것 같다.

### USAGE

```java
// XADD
StreamOperations sop = redisTemplate.opsForStream();
        ObjectRecord<String, T> record = StreamRecords.newRecord()
                .in("test:data")
                .withId("1-1")
                .ofObject(data);
        sop.add(record);

// GETALL
StreamOperations sop = redisTemplate.opsForStream();
        List<ObjectRecord<String, Data>> objectRecords = sop
                .read(Data.class, StreamOffset.fromStart("test:data"));
// 특정 ID 부터 GET ALL 
StreamOperations sop = redisTemplate.opsForStream();
        List<ObjectRecord<String, Data>> objectRecords = sop
                .read(Data.class, StreamOffset.create("test:data", ReadOffset.from("1-20")));


```



### 더 자세한 자료는 [redis-streams.adoc](<https://github.com/spring-projects/spring-data-redis/blob/master/src/main/asciidoc/reference/redis-streams.adoc>)


## 주의 닥친위기
SNAPSHOT 버전이여서 수시로 업데이트를 한다 ... 그래서 가끔 이런 에러가 나오는데 
```
* What went wrong:
Could not resolve all files for configuration ':compileClasspath'.
> Could not find spring-data-parent.pom (org.springframework.data.build:spring-data-parent:2.2.0.BUILD-SNAPSHOT:20190514.161001-386).
  Searched in the following locations:
      https://repo.spring.io/libs-snapshot/org/springframework/data/build/spring-data-parent/2.2.0.BUILD-SNAPSHOT/spring-data-parent-2.2.0.BUILD-20190514.012300-384.pom
> Could not find spring-data-redis.jar (org.springframework.data:spring-data-redis:2.2.0.BUILD-SNAPSHOT:20190514.161221-627).
  Searched in the following locations:
      https://repo.spring.io/libs-snapshot/org/springframework/data/spring-data-redis/2.2.0.BUILD-SNAPSHOT/spring-data-redis-2.2.0.BUILD-20190514.032843-625.jar
> Could not find spring-data-keyvalue.jar (org.springframework.data:spring-data-keyvalue:2.2.0.BUILD-SNAPSHOT:20190514.161156-711).
  Searched in the following locations:
      https://repo.spring.io/libs-snapshot/org/springframework/data/spring-data-keyvalue/2.2.0.BUILD-SNAPSHOT/spring-data-keyvalue-2.2.0.BUILD-20190514.032540-708.jar
> Could not find spring-data-commons-latest.integration.jar (org.springframework.data:spring-data-commons:2.2.0.BUILD-SNAPSHOT:20190514.161041-709).
  Searched in the following locations:
      https://repo.spring.io/libs-snapshot/org/springframework/data/spring-data-commons/2.2.0.BUILD-SNAPSHOT/spring-data-commons-2.2.0.BUILD-20190514.032401-706-latest.integration.jar
> Could not find spring-data-commons.jar (org.springframework.data:spring-data-commons:2.2.0.BUILD-SNAPSHOT:20190514.161041-709).
  Searched in the following locations:
      https://repo.spring.io/libs-snapshot/org/springframework/data/spring-data-commons/2.2.0.BUILD-SNAPSHOT/spring-data-commons-2.2.0.BUILD-20190514.032401-706.jar
```
SNANPSHOT 버전은 저장소에 하나 업데이트하면 하나 지우고 해서 이전 url 기록을 찾다가 못찾는다.<br>
따라서 이런식으로 하면된다.<br>
```
gradlew build --refresh-dependencies
```


