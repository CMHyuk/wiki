# Tokenizer

### Standard

```
GET _analyze
{
  "tokenizer": "standard",
  "text": "THE quick.brown_FOx jumped! @ 3.5 meters."
}
```

**Result**

**["THE", "quick.brown_FOx", "jumped", "3.5", "meters"]**

**Standard** 토크나이저는 공백으로 텀을 구분하면서 "@"과 같은 일부 특수문자를 제거한다. "jumped!"의 느낌표, "meters."의 마침표 처럼 단어 끝에 있는 특수문자는 제거되지만 "quick.brown_FOx" 또는 "3.5" 처럼 중간에 있는 마침표나 밑줄 등은 제거되거나 분리되지 않는다.

---

### Letter

```
GET _analyze
{
  "tokenizer": "letter",
  "text": "THE quick.brown_FOx jumped! @ 3.5 meters."
}
```

**Result**

**["THE", "quick", "brown", "FOx", "jumped", "meters"]**

**Letter** 토크나이저는 알파벳을 제외한 모든 공백, 숫자, 기호들을 기준으로 텀을 분리한다. "quick.brown_FOx" 같은 단어도 "quick", "brown", "FOx" 처럼 모두 분리된 것을 확인할 수 있다.

---

### Whitespace

```
GET _analyze
{
  "tokenizer": "whitespace",
  "text": "THE quick.brown_FOx jumped! @ 3.5 meters."
}
```

**Result**

**["THE", "quick.brown_FOx", "jumped!", "@", "3.5", "meters."]**

**Whitespace** 토크나이저는 스페이스, 탭, 그리고 줄바꿈 같은 공백만을 기준으로 텀을 분리한다. 특수문자 "@" 그리고 "meters." 의 마지막에 있는 마침표도 사라지지 않고 그대로 남아있다.

3개의 토크나이저 중에 **Letter** 토크나이저의 경우 검색 범위가 넓어져서 원하지 않는 결과가 많이 나올 수 있고, 반대로 **Whitespace**의 경우 특수 문자를 거르지 않기 때문에 정확하게 검색을 하지 않으면 검색 결과가 나오지 않을 수 있다. 따라서 보통은 **Standard** 토크나이저를 많이 사용한다.

---

### **UAX URL Email**

주로 사용되는 Standard 토크나이저도 `@`, `/` 같은 특수문자는 공백과 마찬가지로 제거하고 분리한다. 그런데 요즘의 블로그 포스트나 신문기사 같은 텍스트 들에는 이메일 주소 또는 웹 URL 경로 등이 삽입되어 있는 경우가 많다. 이 경우 Standard 토크나이저를 사용하면 이메일 주소등이 정상적으로 인식되지 않아 문제가 될 수 있는데, 이를 방지하기 위해 사용 가능한 것이 **UAX URL Email** 토크나이저이다.

**UAX URL Email** 토크나이저는 이메일 주소와 웹 URL 경로는 분리하지 않고 그대로 하나의 텀으로 저장 한다.

```
GET _analyze
{
  "tokenizer": "uax_url_email",
  "text": "email address is my-name@email.com and website is https://www.elastic.co"
}
```

**Result**

["email", "address", "is", "[my-name@email.com](mailto:my-name@email.com)", "and", "website", "is", "[https://www.elastic.co](https://www.elastic.co/)"]

---

앞에서 살펴본 토크나이저들은 다소 차이는 있지만 기본적으로는 공백을 기준으로 하여 텀들을 분리한다. 분석 할 데이터가 사람이 읽는 일반적인 문장이 아니라 서버 시스템이나 IoT 장비 등에서 수집된 머신 데이터인 경우 공백이 아닌 쉼표나 세로선 같은 기호가 값 항목의 구분자로 사용되는 경우가 종종 있다. 이런 특수한 문자를 구분자로 사용하여 텀을 분리하고 싶은 경우 사용할 수 있는 것이 **Pattern** 토크나이저 입니다.

**Pattern** 토크나이저는 분리할 패턴을 기호 또는 Java 정규식 형태로 지정할 수 있다. 구분자 지정은 `pattern` 항목에 설정한다.

```
PUT pat_tokenizer
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_pat_tokenizer": {
          "type": "pattern",
          "pattern": "/"
        }
      }
    }
  }
}
```

```
GET pat_tokenizer/_analyze
{
  "tokenizer": "my_pat_tokenizer",
  "text": "/usr/share/elasticsearch/bin"
}
```

**Result**

[“user”, “share”, “elasticsearch”, “bin”]

`"pattern": "/"` 같은 단일 기호 외에도 알파벳 대문자를 기준으로 텀을 분리하도록 하는 `"pattern": "(?<=\\p{Lower})(?=\\p{Upper})"`와 같은 정규식(Regular Expression) 으로도 설정이 가능하다.

---

### **Path Hierarchy**

디렉토리나 파일 경로 등은 흔하게 저장되는 데이터다. 앞의 Pattern 토크나이저에서 **"/usr/share/elasticsearch/bin"** 를 실행했을 때는 디렉토리명 들이 각각 하나의 토큰으로 분리 된 것을 확인했다. 이 경우 다른 패스에 있는데 하위 디렉토리 명이 같은 경우 데이터 검색에 혼동이 올 수 있다.

**Path Hierarchy** 토크나이저를 사용하면 경로 데이터를 계층별로 저장해서 하위 디렉토리에 속한 도큐먼트들을 수준별로 검색하거나 집계하는 것이 가능하다.

```
POST _analyze
{
  "tokenizer": "path_hierarchy",
  "text": "/usr/share/elasticsearch/bin"
}
```



**Result**

[”usr”, ”user/share”, ”user/share/elasticsearch”, ”user/share/elasticsearch/bin”]

`delimiter` 항목값으로 경로 구분자를 지정할 수 있다. 디폴트는 `/` 다. 그리고 `replacement` 옵션을 이용해서 소스의 구분자를 다른 구분자로 대치해서 저장하는 것도 가능하다. 그 외의 옵션들은 [공식 도큐먼트](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html)에 있다.

```
PUT hir_tokenizer
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_hir_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "-",
          "replacement": "/"
        }
      }
    }
  }
}
```

```
GET hir_tokenizer/_analyze
{
  "tokenizer": "my_hir_tokenizer",
  "text": [
    "one-two-three"
  ]
}
```

**Result**

[”one”, “one/two”, “one/two/three”]

출처 - https://esbook.kimjmin.net/06-text-analysis/6.5-tokenizer