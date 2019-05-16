## Spring boot profile

배포용 로컬용 등등 프로퍼티를 따로 만들어야한다.


application.properties
application-local.properties
application-prod.properties

정도로 만들고 

1. 첫번째 방법
application.properties 에서 지정해준다
```
spring.profiles.active=local
```

2. 두번째 방법
시작할때 args 로 넘기면된다
```
gradle bootRun --args='--spring.profiles.active=local'
```

아니면 1번 방법으로 적용 시켜놓고 배포 버전일 때만 args 를 주어도 된다.


더 많지만 나중에 .. 

