## match vs term
`match` 와 `term` 쿼리 모두 특정 필드의 내용이 질의어와 일치하는 문서를 찾는데 사용하지만, 일치의 여부를 어떻게 찾는 지가 다르다.

#### match query
`match` 쿼리는 지정한 필드가 질의어와 매치되는 문서를 찾는 쿼리이다. 주로 `text` 타입 필드에 사용된다.
입력된 검색어는 분석기를 통해 토큰화된다. 예를 들어, `Quick Brown Fox`라는 검색어는 `["quick", "brown", "fox"]`로 분할된다.

#### term query
`term` 쿼리는 지정한 필드가 질의어와 정확히 일치하는 문서를 찾는 쿼리이다. 주로 `keyword` 타입 필드에 사용된다.

#### 예시
* `match` 쿼리
```
{
  "query": {
    "match": {
      "description": "Quick Brown Fox"
    }
  }
}
```

`description` 필드를 분석하여, "quick", "brown", "fox"로 분할하고, 이 단어들이 포함된 문서를 검색한다.

* `term` 쿼리  
```
{
  "query": {
    "term": {
      "description": "Quick Brown Fox"
    }
  }
}
```
`description` 필드에서 정확히 "Quick Brown Fox"와 일치하는 문서만 검색한다.