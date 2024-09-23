## Character Filter
아직 토크나이징(Term으로 변환)이 되기 전에 해당 텍스트 전체에 대한 전처리를 수행하는 필터이다. 해당 텍스트에 무언가를 더하거나, 지우거나, 변하는 것이 가능하다.

### HTML Strip
* HTML 태그로 이루어진 텍스트에 해서 해당 태그들을 벗겨주는 캐릭터 필터

```
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    "html_strip"
  ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
```

#### Result
`[ \nI'm so happy!\n ]`  
* 특이점: <p> 태그로 감싼 텍스트가 태그가 사라진 대신에 \n(개행 문자)로 변환

### Mapping
* 특정한 텍스트를 다른 텍스트로 변환

```
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [
        "바보 => 천재"
      ]
    }
  ],
  "text": "바보"
}
```

#### Result
`[ 천재 ]`


### Pattern Replace
* 정규 표현식에 기반하여, 텍스트를 변환하는 필터

```
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)", // (숫자1)-(숫자2) 형태의 패턴에서 "(숫자1)-"만큼 추출
          "replacement": "$1_"   // "(숫자1)-" -> "(숫자1)_"로 치환
    }
  ],
  "text": "123-456-789"
}
```

#### Result
`[ "123_456_789" ]`

출처 : https://esbook.kimjmin.net/06-text-analysis/6.4-character-filter