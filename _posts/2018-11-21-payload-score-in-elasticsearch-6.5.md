---
layout: post
title: elasticsearch 6.5에서 payload scoring 구현하기
author: 하얀눈길
description: null
tags: 'elasticsearch, elasticsearch6.5, payload, payload-score, score, ranking, script-score'
featuredImage: null
img: null
categories: 'elasticsearch'
date: '2018-11-21'
extensions:
  preset: gfm

---


> elastic search가 payload score를 기본으로 지원하지 않는 이유를 잘 모르겠다. 요구하는 사람들이 꽤 있는 것 같은데..
> 아무튼 없으니, 만들어야지...


# Introduction
payload score란 문서의 키워드에 절대적인 값을 주어 검색을 할 때 이 절대적인 값으로 정렬 등 부가적인 scoring이 가능하도록 구성하는 것을 말한다. 이는 루씬에서 아주 예전부터 지원하던 기능으로 알고 있으며, solr같은 경우는 항상 기본 기능으로 제공한다.

헌데, elasticsearch는 아주 예전 버전부터 payload score를 지원하지 않고 있다. 
이상한 것은 색인은 할 수 있도록 해 두었다는 것이다. 
색인된 payload를 어디서 쓰고 있는지 아직 알 수 없지만, 색인만 하고 검색에 사용할 수 없으니 반쪽짜리 기능이 되어 버렸다. 아니 아예 쓸모없는 기능이 되어 버렸다.
대다수의 개발자들이 이런 문제로 인해 플러그인을 개발하여 자체적으로 사용하고 있는 것 같다.
하지만, elastic쪽에서 자신들의 전략인지는 모르겠지만 플러그인 개발조차 아주 어렵게 해 둔 것 같다.
버전업이 되면서 기존 플러그인에 대한 호환성을 전혀 고려해 주지 않기 때문에, 매 버전마다 새로 개발해야 한다.
맘에 들지 않는 정책이지만, 뭐.. 어쩔수 없다. 만들어야지..

**본 문서는 elasticsearch 6.5.0을 기준으로 작성된 plug-in 개발에 대해 설명한다.**
[여기](https://github.com/focuschange/elasticsearch-payload-score)에 전체 소스가 있으니 참고하기 바란다.


# index
색인 데이터를 구성할 때, 특정 키워드에 가중치를 주기 위해 다음과 같이 field를 구성한다고 가정하자

```javascript
{
  "doc_id": "1"
  "key": [
    "yellow|3 blue|1.1 red|1"
  ]
}
```

위 예제는 문서 1번에서 key 필드의 키워드들에 대해서  `yellow`라는 키워드는 가중치를 `3`으로 주고, `blue`는 `1.1`,  `red`는 `1`로 주기 위해 구성해 본 것이다.
우선은 이런 형식의 필드값을 색인하기 위한 tokenizer가 필요하다.
elasticsearch는 이런 토큰을 위해 `delimited_payload`라는 token filter를 제공한다. 

자세한 사항은 [여기](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-delimited-payload-tokenfilter.html)를 참조하자.

이전 버전에서는 `delimited_payload_filter`라는 이름으로 사용됐는데, 바뀌었다. (*이런.. 버전업 되면 또 어떻게 변할지 모른다.*)

settings.analysis.analyzer에 tokenizer를 설정해 두면 된다.

mappings에서는 해당 필드에 term_vector를 설정해야 한다.
payload 값은 term_vector에 들어가게 되는데, term offset 다음에 저장되는 것으로 보여진다.
다음과 같이 지정하면 된다.

```javascript
	"term_vector": "with_positions_offsets_payloads"
```
 
자세한 내용은 [여기](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-vector.html)를 확인하자.
헌데, 공식 문서에 `with_positions_offsets_payloads` 란 것이 없다. (*뭐지??*)
쓰면 된다.

다음은 전체적인 settings, mappings 내용이다.

## elasticsearch setting & mapping


```javascript
{
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas": 0
    },
    "analysis": {
      "analyzer": {
        "payload_analyzer": {
          "type": "custom",
          "tokenizer": "payload_tokenizer",
          "filter": [
            "payload_filter"
          ]
        }
      },
      "tokenizer": {
        "payload_tokenizer": {
          "type": "whitespace",
          "max_token_length": 64
        }
      },
      "filter": {
        "payload_filter": {
          "type": "delimited_payload",
          "encoding": "float"
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "properties": {
        "key": {
          "type": "text",
          "analyzer": "payload_analyzer",
          "term_vector": "with_positions_offsets_payloads",
          "store": true
        }
      }
    }
  }
}
```

# search
이제 우리가 필요로 하는 payload score 검색 기능을 구현하기 위해 plug-in을 만들어야 한다.
script score 기능으로 plug-in을 만들 것이며, 기본적인 소스는 [여기](https://github.com/elastic/elasticsearch/tree/master/plugins/examples/script-expert-scoring)를 참조하면 된다.
공식 샘플소스는 gradle로 되어 있는데, 아마도 모두 클로닝해서 컴파일 하면 에러가 날 것이다.
java 11을 설치해야 한다.

maven dependency는 다음과 같다.
```xml
<dependencies>  
	<dependency>
		<groupId>org.elasticsearch</groupId>  
		<artifactId>elasticsearch</artifactId>  
		<version>6.5.0</version>  
		<scope>provided</scope>  
	</dependency>
</dependencies>
```

6.x버전이 되면서 script plug-in 기본 인터페이스가 ScriptPlugin으로 변경된 것 같다.(*사실 이전 버전은 잘 모른다. 5.x버전과 비교해 보니 완전히 바뀌어 있었다*)


## Plug-in Class 생성
아래는 plug-in 생성을 위한 코드이다.
코드 중에 _SOURCE_VALUE와 _LANG_VALUE를 통해 script score를 명시할 것이다.

PayloadScorePlugin.java
```java
public class PayloadScorePlugin extends Plugin implements ScriptPlugin {  
  
  @Override  
  public ScriptEngine getScriptEngine(Settings settings, Collection<ScriptContext<?>> contexts) {  
      return new WmpScriptEngine();  
  }  
  
  private static class WmpScriptEngine implements ScriptEngine {  
  private final String _SOURCE_VALUE = "payload_score";  
  private final String _LANG_VALUE = "irgroup";  
  
  @Override  
  public String getType() {  
         return _LANG_VALUE;  
  }  
  
  @Override  
  public <T> T compile(String scriptName, String scriptSource, ScriptContext<T> context, Map<String, String> params) {  
  
         if (!context.equals(ScoreScript.CONTEXT)) {  
            throw new IllegalArgumentException(getType()  
                  + " scripts cannot be used for context ["  
				  + context.name + "]");  
		 }  
  
         // we use the script "source" as the script identifier  
		  if (_SOURCE_VALUE.equals(scriptSource)) {  
		         ScoreScript.Factory factory = PayloadScoreFactory::new;  
				 return context.factoryClazz.cast(factory);  
		  }  
	      throw new IllegalArgumentException("Unknown script name " + scriptSource);  
	  }  
  
      @Override  
	  public void close() {  
	         // optionally close resources  
	  }  
   }  
}
```


PayloadScoreFactory.java
```java
public final class PayloadScoreFactory implements ScoreScript.LeafFactory {  
	private final Map<String, Object> params;  
	private final SearchLookup lookup;  
	private final String field;  
	private final String term;  

	public PayloadScoreFactory(Map<String, Object> params, SearchLookup lookup) {  
  
		if (params.containsKey("field") == false) {  
			throw new IllegalArgumentException("Missing parameter [field]");  
		}  
		if (params.containsKey("term") == false) {  
			throw new IllegalArgumentException("Missing parameter [term]");  
		}  
		this.params = params;  
		this.lookup = lookup;  
		field = params.get("field").toString();  
		term = params.get("term").toString();  
	}  
  
	@Override  
	public boolean needs_score() {  
		return false;  
	}  
  
	@Override  
	public ScoreScript newInstance(LeafReaderContext context) throws IOException {  
  
		PostingsEnum postings = context.reader().postings(new Term(field, term), PostingsEnum.PAYLOADS);  
  
		if (postings == null) {  
			return new ScoreScript(params, lookup, context) {  
				@Override  
				public double execute() {  
				return 0.0d;  
			  }  
			};  
		}  
  
		return new ScoreScript(params, lookup, context) {  
			int currentDocid = -1;  
  
			@Override  
			public void setDocument(int docid) {  
	            /*  
	            * advance has undefined behavior calling with 
	            * a docid <= its current docid 
	            */  
	            if (postings.docID() < docid) {  
					try {  
						postings.advance(docid);  
					} catch (IOException e) {  
						throw new UncheckedIOException(e);  
					}  
				}  
	            currentDocid = docid;  
			}  
  
			@Override  
			public double execute() {  
  
				if (postings.docID() != currentDocid) {  
					/*  
					* advance moved past the current doc, so this doc 
					* has no occurrences of the term 
					*/  
					return 0.0d;  
				 }  
				try {  
					int freq = postings.freq();  
					float sum_payload = 0.0f;  
					for(int i = 0; i < freq; i ++)  
					{  
						postings.nextPosition();  
						BytesRef payload = postings.getPayload();  
						if(payload != null) {  
							sum_payload += ByteBuffer.wrap(payload.bytes, payload.offset, payload.length)  
									.order(ByteOrder.BIG_ENDIAN).getFloat();  
						}  
					}  
					
					return sum_payload;  
				} catch (IOException e) {  
					throw new UncheckedIOException(e);  
				}  
			}  
		};  
	}  
}
```

PayloadScorePlugin class는 script score 기능을 추가하기 위한 plugin 설정 정보를 담는다.
실제 score는 PayloadScoreFactory class에서 계산하게 된다.

index field에는 동일한 term이 여러개 있을 수 있으며, 각 term마다 payload값을 다르게 줄 수 있다.
따라서 이를 모두 합산하여 score를 주기 위해 posting을 순회하면서 합산한다.
 
 소스는 짧고 간단하니, 상세한 설명은.. 패스~


# 컴파일 및 설치
컴파일은 다음과 같이 하면 된다.
```bash
maven clean package
```

zip파일이 잘 만들어졌다면 다음과 같이 설치한다.

```bash
# 이전 설치된 플러그인 삭제
$ ES_HOME/bin/elasticsearch-plugin remove payload_score

# 플러그인 등록
$ ES_HOME/bin/elasticsearch-plugin install file:///PROJECT_HOME/releases/payload_score-1.0.0.zip
```
물론 elasticsearch를 중단한 후 재구동해야 한다.
재구동 할 때, 다음 메세지가 출력되면 성공적으로 등록된 것이다.
```
[2018-11-21T22:04:11,793][INFO ][o.e.p.PluginsService     ] [standalone] loaded plugin [payload_score]
```

# example test
테스트를 위해 아래와 같이 간단히 bulk index용 컬렉션을 만들었다.

**test_collection.json**
```javascript
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "1" } }  
{ "key" : ["yellow|3 blue|1.1 red|1" ] }  
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "2" } }  
{ "key" : ["yellow|2 yellow|2.5 blue|1 red|5"] }  
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "3" } }  
{ "key" : ["yellow|10 blue|2 red|4.3" ] }  
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "4" } }  
{ "key" : ["dark_yellow|10 blue|3 red|4.2" ] }  
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "5" } }  
{ "key" : ["yellow|102020.95 blue red|1" ] }
```

색인 생성 및 벌크로드를 아래 스크립트로 간단히 생성하였으니, 유용하게.. (*쿨럭~*)

**create_index.sh**
```bash
#!/bin/bash  
  
OPTIND=1 # Reset in case getopts has been used previously in the shell.  
  
INDEX_NAME='payload_test'  
ES_HOST='localhost:9200'  
SET_MAP='set-map.json'  
  
function help {  
  echo "Usage : $0 [options]"  
  echo "    -h|-?                 help"  
  echo "    -H {host:port}        host:port. \"localhost:9200\" is default "  
  echo "    -i {index}            index"  
  echo "    -s {set-map_file}     setting & mapping json file"  
  echo ""  
}  
  
if [ "$#" -eq 0 ]; then  
  help  
  exit;  
fi  
  
while getopts "H:i:s:" opt; do  
 case "$opt" in  
  \h|\?)  
        help  
  exit 0  
  ;;  
  H) ES_HOST=$OPTARG  
        ;;  
  i) INDEX_NAME=$OPTARG  
        ;;  
  s) SET_MAP=$OPTARG  
        ;;  
  :)  
        echo "Option -$OPTARG requires an argument." >&2  
  echo ""  
  help  
  exit 0  
  ;;  
 esacdone  
  
shift $((OPTIND-1))  
[ "$1" = "--" ] && shift  
  
echo '* remove old index'  
curl -X DELETE http://${ES_HOST}/${INDEX_NAME}?pretty  
echo  
echo  
  
  
echo '* create index'  
curl -H 'Content-Type: application/json' -XPUT http://${ES_HOST}/${INDEX_NAME}?pretty --data-binary @${SET_MAP}  
echo  
echo  
  
  
echo '* index info'  
curl -XGET http://${ES_HOST}/${INDEX_NAME}?pretty  
echo  
  
echo '* bulk insert'  
curl -H 'Content-Type: application/json' -X POST http://${ES_HOST}/_bulk?pretty --data-binary @test_collection.json  
echo
```

아래와 같이 사용하면 된다.

```bash
./create_index.sh -i payload-test -s payload_set-map.json
```

정상적이라면 localhost:9200 클러스터에 payload-test index가 생성되어 있을 것이다.
다음과 같이 test_collection을 색인해 보자

```bash
curl -H 'Content-Type: application/json' -X POST http://localhost:9200/_bulk?pretty --data-binary @test_collection.json
```

색인된 payload 값을 확인해 보자

```bash
curl -H 'Content-Type: application/json' -XGET localhost:9200/payload-test/_doc/1/_termvectors?pretty
```

아마도 아래처럼 나오게 될 것이다.
```bash
{
  "_index" : "payload-test",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "found" : true,
  "took" : 0,
  "term_vectors" : {
    "key" : {
      "field_statistics" : {
        "sum_doc_freq" : 12,
        "doc_count" : 4,
        "sum_ttf" : 13
      },
      "terms" : {
        "blue" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 1,
              "start_offset" : 9,
              "end_offset" : 17,
              "payload" : "P4zMzQ=="
            }
          ]
        },
        "red" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 2,
              "start_offset" : 18,
              "end_offset" : 23,
              "payload" : "P4AAAA=="
            }
          ]
        },
        "yellow" : {
          "term_freq" : 1,
          "tokens" : [
            {
              "position" : 0,
              "start_offset" : 0,
              "end_offset" : 8,
              "payload" : "QEAAAA=="
            }
          ]
        }
      }
    }
  }
}
```

tokens에 보면 payload 값이 보인다.
문서는 float으로 구성했지만, 엔진에서 base64로 인코딩한다. 

자.. 이제 대망의 payload score 검색을 해 보자

```bash
$ curl -H 'Content-Type: application/json' -X POST 'localhost:9200/payload-test/_search?pretty' -d '
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "key": "yellow"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": {
                "source": "payload_score",
                "lang" : "irgroup",
                "params": {
                    "field": "key",
                    "term": "yellow"
                }
            }
          }
        }
      ]
    }
  }
}
'
```

fuction_score query를 사용하여 검색한다. 
script_score에서 source, lang을 프로그램에서 설정한 값으로 입력하고, params 부분에서는 검색대상 필드와 검색할 키워드를 입력한다.

결과는 다음과 같이 나올 것이다.

```javascript
{
  "took" : 73,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 102020.95,
    "hits" : [
      {
        "_index" : "payload-test",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 102020.95,
        "_source" : {
          "key" : [
            "yellow|102020.95 blue red|1"
          ]
        }
      },
      {
        "_index" : "payload-test",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 10.0,
        "_source" : {
          "key" : [
            "yellow|10 blue|2 red|4.3"
          ]
        }
      },
      {
        "_index" : "payload-test",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 4.5,
        "_source" : {
          "key" : [
            "yellow|2 yellow|2.5 blue|1 red|5"
          ]
        }
      },
      {
        "_index" : "payload-test",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 3.0,
        "_source" : {
          "key" : [
            "yellow|3 blue|1.1 red|1"
          ]
        }
      }
    ]
  }
}
```

잘 나온다. 성공이다~
내용을 좀 더 훓어보면, `yellow`라는 키워드로 검색할 때, `score` 값이 `payload` 값으로 나오는 것을 확인 할 수 있다.
이때, `_id` = `2`인 문서는 `yellow` 키워드가 2개이고, 각각의 `payload` 값을 합산한 것이 `score`에 반영된 것을 알 수 있다.


# 참조

 - https://www.elastic.co/guide/en/elasticsearch/plugins/current/plugin-authors.html
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-engine.html
 - https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-delimited-payload-tokenfilter.html
 - https://github.com/elastic/elasticsearch/tree/master/plugins/examples/script-expert-scoring
 - https://github.com/focuschange/elasticsearch-payload-score
 


