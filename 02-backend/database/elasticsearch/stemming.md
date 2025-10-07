### Nori
한글은 형태의 변형이 매우 복잡한 언어다.  
특히 복합어, 합성어 등이 많아 하나의 단어도 여러 어간으로 분리해야 하는 경우가 많아 한글을 형태소 분석을 하려면 반드시 한글 형태소 사전이 필요하다.

### 설치
* 설치  
`$ bin/elasticsearch-plugin install analysis-nori`  

* 제거  
`$ bin/elasticsearch-plugin remove analysis-nori`

### nori_tokenizer
Nori는 nori_tokenizer 토크나이저와 nori_part_of_speech, nori_readingform 토큰 필터를 제공한다.  
먼저 nori_tokenizer 토크나이저를 사용해서 한글을 간단하게 테스트 할 수 있다.  
다음은 standard와 nori_tokenizer 를 비교해서 "동해물과 백두산이" 를 분석한 예제다.  
analysis-nori 플러그인이 설치되어 있어야 한다.

**Standard**
```
GET _analyze
{
  "tokenizer": "standard",
  "text": [
    "동해물과 백두산이"
  ]
}
```

**Result**  
["동해물과", "백두산이"]

**Nori**
```
GET _analyze
{
  "tokenizer": "nori_tokenizer",
  "text": [
    "동해물과 백두산이"
  ]
}
```

**Result**  
["동해", "물", "과", "백두", "산", "이"]  

Standard 토크나이저는 공백 외에 아무런 분리를 하지 못했지만, nori_tokenizer는 한국어 사전 정보를 이용해 "token" : "동해", "token" : "산" 같은 단어을 분리 한 것을 확인할 수 있다.  

nori_tokenizer에는 다음과 같은 옵션들이 있다.  

* user_dictionary : 사용자 사전이 저장된 파일의 경로를 입력한다. 
* user_dictionary_rules : 사용자 정의 사전을 배열로 입력한다. 
* decompound_mode : 합성어의 저장 방식을 결정한다. 다음 3개의 값을 사용 가능하다. 
  * none : 어근을 분리하지 않고 완성된 합성어만 저장한다. 
  * discard (디폴트) : 합성어를 분리하여 각 어근만 저장한다. 
  * mixed : 어근과 합성어를 모두 저장한다.  

user_dictionary는 다른 애널라이저들과 마찬가지로 config 디렉토리의 상대 경로를 입력하며 변경시 인덱스를 _close / _open 하면 반영된다. 
사전의 단어들에는 우선순위가 있으며 문장 "동해물과" 에서는 "동해" 가 가장 우선순위가 높아 "동해" 가 먼저 추출되고 다시 "물" 그리고 "과" 가 추출되어 "동해"+"물"+"과" 같은 형태가 된다. 
user_dictionary 경로에 있는 사전 파일이나 user_dictionary_rules 설정 값에 단어만 나열하면 이 단어들을 가장 우선으로 추출한다.  

**user_dictionary_rules**
```
PUT my_nori
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_nori_tokenizer": {
          "type": "nori_tokenizer",
          "user_dictionary_rules": [
            "해물"
          ]
        }
      }
    }
  }
}
```

```
GET my_nori/_analyze
{
  "tokenizer": "my_nori_tokenizer",
  "text": [
    "동해물과"
  ]
}
```

**Result**  
["동", "해물", "과"]  

이렇게 사용자 사전에 "해물" 이라는 단어를 추가하면 "동해물과" 는 "동"+"해물"+"과" 로 분석이 되어 "해물"로 검색이 된다.  

**decompound_mode**
```
PUT my_nori
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "nori_none": {
          "type": "nori_tokenizer",
          "decompound_mode": "none"
        },
        "nori_discard": {
          "type": "nori_tokenizer",
          "decompound_mode": "discard"
        },
        "nori_mixed": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed"
        }
      }
    }
  }
}
```

**none**
```
GET my_nori/_analyze
{
  "tokenizer": "nori_none",
  "text": [ "백두산이" ]
}
```

**Result**  
["백두산", "이"]  

**nori_discard**  
```
GET my_nori/_analyze
{
  "tokenizer": "nori_discard",
  "text": [ "백두산이" ]
}
```  

**Result**  
["백두", "산", "이"]  

**nori_mixed**  
```
GET my_nori/_analyze
{
  "tokenizer": "nori_mixed",
  "text": [ "백두산이" ]
}
```  

**Result**  
["백두산", "백두", "산", "이"]  

각 설정에 따라 어근을 분리하거나 분리하지 않거나 모두 저장하는 것을 확인할 수 있다.  
decompound_mode 의 디폴트 값은 discard다.