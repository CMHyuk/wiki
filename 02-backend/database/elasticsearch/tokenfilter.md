## TokenFilter

### **Lowercase, Uppercase**

영어나 유럽어 기반의 텍스트는 대소문자가 있어 검색할 때는 대소문자에 상관 없이검색이 가능하도록 처리 해 주어야 한다. 보통은 텀들을 모두 소문자로 변경하여 저장하는데 이 역할을 하는 것이 **Lowercase** 토큰 필터다. **Lowercase** 토큰 필터는 거의 모든 텍스트 검색 사례에서 사용되는 토큰 필터이다.

**Uppercase** 토큰 필터는 모든 텀을 대문자로 변경하는 것이며 Lowercase 와 동일하게 설정한다.

### **Lowercase**

```
GET _analyze
{
  "filter": [ "lowercase" ],
  "text": [ "Harry Potter and the Philosopher's Stone" ]
}
```

**Result**  
[”harry potter and the philosopher's stone”]

### **Uppercase**

```
GET _analyze
{
  "filter": [ "uppercase" ],
  "text": [ "Harry Potter and the Philosopher's Stone" ]
}
```

**Result**   
["HARRY POTTER AND THE PHILOSOPHER'S STONE”]

- - -

### Stop

블로그 포스트나 뉴스 기사 같은 글에는 검색에서는 큰 의미가 없는 조사나 전치사 등이 많다.  
영문에서도 **the**, **is**, **a** 같은 단어들은 대부분 검색어로 쓰이지 않는데 이런 단어를 한국어로는 **불용어**, 영어로는 **stopword**라고 한다.  

**Stop** 토큰 필터를 적용하면 불용어에 해당되는 텀들을 제거한다.  
`stopwords` 항목에 불용어로 지정할 단어들을 배열 형태로 나열하거나 `"_english_"`, `"_german_"` 같이 언어를 지정해서 해당 언어팩에 있는 불용어를 지정할 수도 있다.  
지원되는 언어팩은 [공식 도큐먼트](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stop-tokenfilter.html)에서 확인할 수 있으며 한, 중, 일어 등은 별도의 형태소 분석기를 사용해야 한다.  
불용어 목록을 별도의 텍스트 파일로 저장하고 저장된 파일 경로를 `stopwords_path` 항목의 값으로 지정하여 사용하는 것도 가능하다.  

다음은 **my_stop** 인덱스에 **"in"**, **"the"**, **"days"** 를 불용어로 처리하는 **my_stop_filter** 라는 이름의 `stop` 토큰필터를 정의하고 `lowercase` 필터와 함께 **"Around the World in Eighty Days"** 문장을 분석 하는 예제이다.

```
PUT my_stop
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stop_filter": {
          "type": "stop",
          "stopwords": [
            "in",
            "the",
            "days"
          ]
        }
      }
    }
  }
}
```

```
GET my_stop/_analyze
{
  "tokenizer": "whitespace",
  "filter": [
    "lowercase",
    "my_stop_filter"
  ],
  "text": [ "Around the World in Eighty Days" ]
}
```

**Result**  
[”Aoround”, “World”, “Eighty”]

이번에는 불용어 "in", "the", "eighty"를 my_stop_dic.txt 파일 안에 저장하고 이 파일을 읽어들여 동일한 문장을 분석 해 보는 예제다.  
불용어는 모두 줄바꿈으로 입력해야 하며 사전 파일 경로는 elasticsearch 의 config 디렉토리를 기준으로 상대 경로를 지정해야 하며 텍스트 인코딩은 반드시 UTF-8 로 되어 있어야 한다.

```
PUT my_stop
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stop_filter": {
          "type": "stop",
          "stopwords_path": "user_dic/my_stop_dic.txt"
        }
      }
    }
  }
}
```

```
GET my_stop/_analyze
{
  "tokenizer": "whitespace",
  "filter": [
    "lowercase",
    "my_stop_filter"
  ],
  "text": [ "Around the World in Eighty Days" ]
}
```

**Result**  
[”Around”, “World”, “Days”]

- - -

### Synonym
검색 서비스에 따라서 **동의어** 검색을 제공해야 하는 경우가 있다. 예를 들면 클라우드 서비스 관련 정보를 검색하는 시스템에서 "AWS" 라는 단어를 검색했을 때 "Amazon" 또는 한글 "아마존" 도 같이 검색을 하도록 하면 관련된 정보를 더 많이 찾을 수 있을 것이다. 
이 때 **Synonym** 토큰 필터를 사용하면 텀의 동의어 저장이 가능하다.  

동의어를 설정하는 옵션은 `synonyms` 항목에서 직접 동의어 목록을 입력하는 방법과 동의어 사전 파일을 만들어 `synonyms_path` 로 지정하는 방법이 있다. 동의어 사전 명시 규칙에는 다음의 것들이 있다.  
- `"A, B => C"` : 왼쪽의 A, B 대신 오른쪽의 C 텀을 저장한다. A, B 로는 C의 검색이 가능하지만 C 로는 A, B 가 검색되지 않는다.
- `"A, B"` : A, B 각 텀이 A 와 B 두개의 텀을 모두 저장한다. A 와 B 모두 서로의 검색어로 검색이 된다.  

```
PUT my_synonym
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_syn": {
          "tokenizer": "whitespace",
          "filter": [
            "lowercase",
            "syn_aws"
          ]
        }
      },
      "filter": {
        "syn_aws": {
          "type": "synonym",
          "synonyms": [
            "amazon => aws"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "analyzer": "my_syn"
      }
    }
  }
}
```

```
PUT my_synonym/_doc/1
{ "message" : "Amazon Web Service" }
PUT my_synonym/_doc/2
{ "message" : "AWS" }
```

```
GET my_synonym/_termvectors/1?fields=message
```

**Result**  
["Amazon Web Service"가 amazon 대신 "aws", "web", "service"]

- - -
### Unique
"white fox, white rabbit, white bear" 같은 문장을 분석하면 "white" 텀은 총 3번 저장이 된다.  
역 색인에는 텀이 1개만 있어도 텀을 포함하는 도큐먼트를 가져올 수 있기 때문에 중복되는 텀들은 삭제해도 된다.  
unique 토큰 필터를 사용해서 중복되는 텀 들은 하나만 저장하도록 할 수 있다.  

```
GET _analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": [
    "white fox, white rabbit, white bear"
  ]
}
```

```
GET _analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "unique"
  ],
  "text": [
    "white fox, white rabbit, white bear"
  ]
}
```

**Result**  
["white", "fox", "rabbit", "bear"]

 출처 -  https://esbook.kimjmin.net/06-text-analysis/6.6-token-filter