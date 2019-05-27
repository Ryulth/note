## Elasticsearch with Nori sBasic Settings

한글은 조사나 형용사가 많아 검색하는데 어려움이 있다. 그래서 형태소 분석기를 사용해서 검색을 도와주는데 Nori 라고 공식 한글 형태소 분석기가 있다. 공식지원인 만큼 설치가 매우 간단하다.

[Nori 설치](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html)

[Elasticsearch 설치](<https://www.elastic.co/kr/downloads/elasticsearch>)

설치를 한 후에 Elasticsearch(기본 9200 포트)에 요청을 보내보자

GET `http://localhost:9200/_analyze `

```json
{
	"analyzer" : "nori",
	"text" : "아버지가 방에 들어가신다"
}
```

curl, postman, Kibana 를 사용하든 어떻게든 보내보자

```json
{
    "tokens": [
        {
            "token": "아버지",
            "start_offset": 0,
            "end_offset": 3,
            "type": "word",
            "position": 0
        },
        {
            "token": "방",
            "start_offset": 5,
            "end_offset": 6,
            "type": "word",
            "position": 2
        },
        {
            "token": "들어가",
            "start_offset": 8,
            "end_offset": 11,
            "type": "word",
            "position": 4
        }
    ]
}
```

응답이 이런식으로 오면 설치가 된것

실제 한글 검색에서 사용하려면 조금의 설정이 필요하다

POST `http://localhost:9200/nori_test/user`

```json
{
	"name" : "아버지",
	"action" : "아버지가 가방에 들어가신다"
}
```

데이터를 넣어놓고

GET `http://localhost:9200/nori_test/user/_search` 하면 결과는 안나온다.

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 0,
        "max_score": null,
        "hits": []
    }
}
```

 토큰 설정 및 mapping 설정을 안해줬기때문. `아버지가` 라고 검색하면 나온다. 이러면 검색에 쓸모가 없기 때문에 설정을 해줘야한다.

일단 사용중인 index 를 지워주자 mapping 해줘야한는데 한번 생긴 mapping 은 변경하기 힘들다 mapping update 는 귀찮음으로 일단 지워버리자 [Mapping 변경](<https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html>)

DELETE `http://localhost:9200/nori_test`

지워버리고 새로 설정파일을 주자 

PUT `http://localhost:9200/nori_test`

```json
{
    "settings": {
        "index": {
            "analysis": {
                "tokenizer": {
                    "nori_user_dict": {
                        "type": "nori_tokenizer",
                        "decompound_mode": "mixed",
                        "user_dictionary": "userdict_ko.txt"
                    }
                },
                "analyzer": {
                    "korean": {
                        "type": "custom",
                        "tokenizer": "nori_user_dict"
                    }
                }
            }
        }
    },
    "mappings": {
        "user": {
            "properties": {
                "action" : {
                	"type":"text",
                	"analyzer": "korean"
                }
            }
        }
    }
}
```

`userdict_ko.txt` 이건 나중에 커스텀한 단어가 필요할 경우에 사용한다 config 폴더 밑에 만들어주면 된다.

`tokenizer` 만들고 그걸로 `analyzer` 를 만든다. 만든걸로 mapping 해주는 것이다.

일단 설정했으니 index 에 `analyzer` 가 설정 됬나 확인 해보자

GET `http://localhost:9200/nori_test/_analyze`

```json
{
	"analyzer" : "korean",
	"text" : "아버지가 방에 들어가신다"
}
```

결과

```json
{
    "tokens": [
        {
            "token": "아버지",
            "start_offset": 0,
            "end_offset": 3,
            "type": "word",
            "position": 0
        },
        {
            "token": "가",
            "start_offset": 3,
            "end_offset": 4,
            "type": "word",
            "position": 1
        },
        {
            "token": "방",
            "start_offset": 5,
            "end_offset": 6,
            "type": "word",
            "position": 2
        },
        {
            "token": "에",
            "start_offset": 6,
            "end_offset": 7,
            "type": "word",
            "position": 3
        },
        {
            "token": "들어가",
            "start_offset": 8,
            "end_offset": 11,
            "type": "word",
            "position": 4
        },
        {
            "token": "신다",
            "start_offset": 11,
            "end_offset": 13,
            "type": "word",
            "position": 5,
            "positionLength": 2
        },
        {
            "token": "시",
            "start_offset": 11,
            "end_offset": 13,
            "type": "word",
            "position": 5
        },
        {
            "token": "ㄴ다",
            "start_offset": 11,
            "end_offset": 13,
            "type": "word",
            "position": 6
        }
    ]
}
```

결과가 잘 나올 것이다. 이러면 설정 완료다.

설정이 완료 하였으니 아까 처음처럼 데이터를 넣어보자.

POST `http://localhost:9200/nori_test/user`

```json
{
	"name" : "아버지",
	"action" : "아버지가 가방에 들어가신다"
}
```

넣어준 후 이번엔 `아버지` 라는 단어로 검색해보자

GET `http://localhost:9200/nori_test/user/_search`

```json
{
    "query": {
        "match": {
            "action": "아버지"
        }
    }
}
```

결과는 

```json
{
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.3031859,
        "hits": [
            {
                "_index": "nori_test",
                "_type": "user",
                "_id": "Boh1-GoBEmgWXPhrCvBY",
                "_score": 0.3031859,
                "_source": {
                    "name": "아버지",
                    "action": "아버지가 가방에 들어가신다"
                }
            }
        ]
    }
}
```

action 에 대해 아버지란 단어가 포함된 데이터를 가져 올 수 있다.

그냥 되나 안되나 학습용으로 연습해본 결과이고 실제 작업시에는 `setting` 의 정교함이 필요할 것 같다.

그리고 `mapping` 만 잘 변경할 수 있도록 해야한다.

**Multiple mapping types are not supported in indices created in 6.0**

근데 Elasticsearch 6.0 부터는 한 인덱스에 한 타입이란다. 이점 조심해서 세팅해야 할 것이다.