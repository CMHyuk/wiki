#### Index
* Document를 저장하는 논리적 단위 
* RDB의 테이블과 같은 개념 
* Index 명은 소문자 + 일부 특수문자, 255 Byte 이하
제외 : \ / * ? “ < > | # , <공백>

#### Document
* 실제 데이터를 저장하는 단위 
* JSON 형태, 여러 field, value 가 존재 
* RDB의 record 과 같은 개념 
* 각 Document 는 하나의 index에 포함
같은 index 의 Document 도 field 가 다를 수 있음

