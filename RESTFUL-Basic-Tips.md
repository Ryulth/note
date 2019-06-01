# RESTFUL Basic Tips 

* 이전에 RestApi 를 작성할때 간과했던 문제들을 다시 짚어 보았다.

## RESTful 이란

* REST는 Representational State Transfer라는 용어의 약자로서 웹의 장점을 최대한 활용할 수 있는 아키텍처
* 요즘은 다양한 클라이언트가 많아져 하나의 서버로 여러대의 클래이언트를 대응하도록 해야합니다..
* URL -> URI 를 사용하는 시대로 변화해 왔기 때문에 사용자가 의미를 쉽게 알 수 있는 식별자를 만드는 것이 중요합니다.

## GET or POST?

### POST 방식이 GET 방식보다 보안측면에서 더 좋은가?

* POST 든 GET 이든 암호화 없이 보내면 보안은 똑같습니다. 단지 URL 에 표시되어 바로 보인다는점 빼곤 차이없습니다. URL 에 표시하고 싶지 않으면 Request header 에 담아서 보내면 됩니다.
* 둘중 어떤 것을 사용하든 암호화를 해야 보안이 됩니다.

### GET방식이 POST방식보다 속도가 빠르다 왜?

* GET 방식의 요청은 캐싱을 하기 때문에 빠른것이다.

### 그럼 언제 GET 쓰고 POST 를 쓰나?

> GET은 가져오는 것이고 POST는 수행하는 것입니다.

* GET은 Idempotent, POST는 Non-idempotent
* GET 설계원칙 : 서버에게 동일한 요청을 여러 번 전송하더라도 동일한 응답이 돌아와야 한다는 것 -> 따라서 조회 할때 사용
* POST 설계원칙 : 서버에게 동일한 요청을 여러 번 전송해도 응답은 항상 다를 수 있다. -> 따라서 서버의 데이터를 변경시킬 때 사용

> #### POST는 생성, 수정, 삭제에 사용할 수 있지만, 생성에는 POST, 수정은 PUT 또는 PATCH, 삭제는 DELETE가 더 용도에 맞는 메소드라고 할 수 있습니다.

## 효과적인 URI 설계하기

### CRUD 는 URI 에 사용하면 안된다.

가장 쉽게 실수(?) 할 수 있는 부분이라고 생각합니다. 프로그래밍 측면에서 함수를 구성하다보면 직관적인 네이밍을 중요하게 생각합니다. 그러다보니 URI 도 직관적으로 구성하려고 했습니다

* 예를들면

```
GET /posts/all
GET /posts/12/show
POST /posts/add
POST /posts/12/update
GET /posts/delete/11
```

이런식으로 GET 과 POST 만 사용해서 구성하는 경우가 많았을 것입니다. 저도 저렇게 작성 했었습니다.

과거에는 GET, POST 만 사용 가능하다고 하였으니 지금부터는 RESTful 하게 작성할 수 있어야 할 것입니다. 이제는 CRUD 기능은 URI 에 작성하지 않도록 해야 합니다.

* HTTP METHOD의 알맞은 역할

| CRUD | METHOD | 설명 |
| ---- | ------ | --- |
| Create | POST | 리소스를 생성하는데 주력 |
| Read or Retrieve | GET | 리소스를 조회하고 정보를 가져오는데 주력 |
| Update | PUT | 리소스를 수정하는데 주력 |
| Delete | DELETE | 리소스를 삭제하는데 주력 |

위의 예시를 다시 작성해 본다면

* API Refactoring

```
GET /posts
GET /posts/12
POST /post
PUT /posts/12
DELETE /posts/12
```

METHOD 명으로 어떠한 동작이 진행되는지 알 수 있도록 할 수 있습니다.

### 되도록 소문자를 사용하자

* 대소문자를 구별하기 때문에 사용자의 실수를 유발하기 쉽기 때문
* 만약 도메인을 의미있게 사용하려고 대문자를 사용한다면 최소한 대소문자가 구분된다는걸 알고 사용해야 합니다.

```
http://www.example.com/board/soccer
http://www.example.com/board/Soccer
```

이 두개의 URI 는 다른 리소스를 가르킵니다.

### 하이픈(-)을 가독성을 높이는데 사용하자

* 경로가 짧을 수록 좋지만 길어질 경우 언더바 대신 하이픈을 사용하기로 합니다
* 경로에 띄어쓰기가 들어가는 경우 %20이 들어가 가독성이 매우 떨어진다.
* 언더바는 하이퍼링크나 밑줄이 쳐진경우 가려질 경우가 많기 때문에 명확하게 보이기 위해서 하이픈을 사용하기로 합니다.
http://www.example.com/board/soccer_study -> 언더바가 보이지 않음
http://www.example.com/board/soccer-study -> 경로를 명확하게 확인 가능

### 확장자를 사용하지 않는 방향으로 합니다.

* 확장자를 사용하지 않으면 매우 유연해집니다.

A 라는 클라이언트는 txt 형태로 받아야만 하고 B 라는 클라이언트는 csv 형태로 받아야 한다면 기존에는 두가지 URI 를 만들어 사용해왔습니다.

http://www.example.com/score.txt
http://www.example.com/score.csv

이 방식에서는 만약 json 형태를 추가로 요구한다면 URI 를 추가로 생성해야 합니다. 요즘 다양한 클라이언트가 존재하는 만큼 요청을 보낸 클라이언트를 판단하고 리소스를 전달해주어야합니다.

* 이걸 어떻게 판단하는지 의문이 들 수 있습니다. 이것은 결론적으론 Accept header 를 이용해서 판단합니다. [HTTP 공통 & 요청 헤더](https://www.zerocho.com/category/HTTP/post/5b3ba2d0b3dabd001b53b9db)
* http://www.example.com/score 한개의 URI 를 가지고 대응 가능하도록 할 수 있습니다

간단하게 작성해본다면 이런식의 결과를 얻을 수 있다.

```
GET /score
Host : example.com
Accept : text/plain

Response : score.txt
```

```
GET /score
Host : example.com
Accept : text/csv

Response : score.csv
```

```
GET /score
Host : example.com
Accept : application/json

Response : score.json
```

### 자원을 표현할때 Collection과 Document

* Collection 은 집합 객체 리스트 같은 존재
* Document 는 단순한 정보를 가져오는 객체

```
GET /posts/12/author
```

* posts가 Collection에 해당되며 복수로 표현한다.
* author 은 정보를 가져오려고 하는 것임으로 단수형으로 표현한다.

### URI 마지막 문자는 슬래시(/) 를 사용하지 않는다.

* URI 가 다르면 리소스가 다르다는 것입니다. / 를 사용한다는 것은 다음 단계가 존재한다고 판단이 가능합니다. 따라서 혼동을 주지 않기 위해 마지막에는 슬래시(/)를 사용하지 않도록 합니다.

```
http://www.example.com/board/soccer/ (X)
http://www.example.com/board/soccer  (O)
```
### 마치며
글을 쓰다 보니 정확한 의미를 알지 못한 채 POST가 좋다고 POST 를 사용해서 API 를 작성 했던 것이 부끄러워졌다.<br>
정확한 이해를 바탕으로 사용한는 것이 중요하다고 생각이 됩니다.
