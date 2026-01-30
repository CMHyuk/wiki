## 텍스트 분석 - Text Analysis
Elasticsearch에 저장되는 도큐먼트는 모든 문자열(text) 필드 별로 역 인덱스를 생성한다.  

![img.png](../../assets/images/invertedindex3.png)

Elasticsearch는 문자열 필드가 저장될 때 데이터에서 검색어 토큰을 저장하기 위해 여러 단계의 처리 과정을 거친다. 
이 전체 과정을 텍스트 분석(Text Analysis) 이라고 하고 이 과정을 처리하는 기능을 애널라이저(Analyzer) 라고 한다. 
Elasticsearch의 애널라이저는 0~3개의 캐릭터 필터(Character Filter)와 1개의 토크나이저(Tokenizer), 그리고 0~n개의 토큰 필터(Token Filter)로 이루어진다.

![img.png](../../assets/images/analyzer.png)

텍스트 데이터가 입력되면 가장 먼저 필요에 따라 전체 문장에서 특정 문자를 대치하거나 제거하는데 이 과정을 담당하는 기능이 캐릭터 필터다.  
다음으로는 문장에 속한 단어들을 텀 단위로 하나씩 분리 해내는 처리 과정을 거치는데 이 과정을 담당하는 기능이 토크나이저 이다. 토크나이저는 반드시 1개만 적용이 가능하다. 다음은 whitespace 토크나이저를 이용해서 공백을 기준으로 텀 들을 분리 한 결과다.

![img.png](../../assets/images/tokenizer1.png)
다음으로 분리된 텀들을 하나씩 가공하는 과정을 거치는데 이 과정을 담당하는 기능이 토큰 필터다. 토큰 필터는 0개 부터 여러 개를 적용할 수 있다.  
여기서는 먼저 lowercase 토큰 필터를 이용해서 대문자를 모두 소문자로 바꿔준다. 이렇게 하면 대소문자 구별 없이 검색이 가능하게 된다. 대소문자가 일치하게 되어 같은 텀이 된 토큰들은 모두 하나로 병합이 된다.

![img.png](../../assets/images/tokenizer2.png)

이제 역 인덱스는 아래와 같이 변경된다.

![img.png](../../assets/images/tokenizer3.png)

텀 중에는 검색어로서의 가치가 없는 단어들이 있는데 이런 단어를 불용어(stopword) 라고 한다. 보통 a, an, are, at, be, but, by, do, for, i, no, the, to … 등의 단어들은 불용어로 간주되어 검색어 토큰에서 제외된다.  
stop 토큰 필터를 적용하면 우리가 만드는 역 인덱스에서 the가 제거된다.

![img.png](../../assets/images/tokenizer4.png)

이제 형태소 분석 과정을 거쳐서 문법상 변형된 단어를 일반적으로 검색에 쓰이는 기본 형태로 변환하여 검색이 가능하게 한다. 
영어에서는 형태소 분석을 위해 snowball 토큰 필터를 주로 사용하는데 이 필터는 ~s, ~ing 등을 제거한다. 
그리고 happy, lazy 와 같은 단어들은 happiness, laziness와 같은 형태로도 사용되기 때문에 ~y 를 ~i 로 변경한다. 
snowball 토큰 필터를 적용하고 나면 jumps와 jumping은 모두 jump로 변경되고, 동일하게 jump 로 되었기 때문에 하나의 텀으로 병합된다.

![img.png](../../assets/images/tokenizer5.png)

필요에 따라서는 동의어를 추가 해주기도 한다. synonym 토큰 필터를 사용하여 quick 텀에 동의어로 fast를 지정하면 fast 로 검색했을 때도 같은 의미인 quick 을 포함하는 도큐먼트가 검색되도록 할 수 있다. 
AWS 와 Amazon을 동의어로 놓아 amazon을 검색해도 AWS 를 찾을 수 있게 하는 등 실제로도 사용되는 사례가 많다.

![img.png](../../assets/images/tokenizer6.png)

### 요약
* 검색어 토큰을 저장하기 위해 여러 단계의 처리 **과정** -> **텍스트 분석(Text Analysis)**
* 검색어 토큰을 저장하기 위해 여러 단계의 처리 **기능** -> **애널라이저(Analyzer)**
* 애널라이저는 0~3개의 캐릭터 필터(Character Filter), 1개의 토크나이저(Tokenizer), 0~n개의 토큰 필터(Token Filter)로 구성
* 전체 문장에서 특정 문자를 대치하거나 제거 과정 담당 기능 -> **캐릭터 필터**
* 문장에 속한 단어들을 텀 단위로 하나씩 분리 해내는 처리 과정 담당 기능 -> **토크나이저(반드시 1개)**
* 분리된 텀들을 하나씩 가공하는 과정 -> **토큰 필터**

출처 - https://esbook.kimjmin.net/06-text-analysis/6.2-text-analysis