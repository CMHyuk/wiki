### String Collection
* Redis는 기본적으로 다양한 타입을 지원한다. 
* SET : 데이터를 그냥 문자열로 저장하는 방법, 이진 데이터도 포함되기 때문에 이미지도 저장 가능 
* SETNX : 키가 존재하지 않는 경우에 대해서만 새로운 키를 저장, 일반적인 SET에 비해 성능 차이가 조금 더 우수함 
* MEST : Redis에서의 Bulk Write 방법, 네트워크 IO를 줄여주기 때문에 성능적으로 우수 
  * NX와 M을 조합하여 MSETNX로도 활용 가능  
* INCRBY, DECRBY는 데이터가 정수형인 경우에 대해서 유효하게 동작 가능