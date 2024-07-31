**과거**

General 헤더 : 메시지 전체에 적용되는 정보

Request 헤더 : 요청 정보

Response 헤더 : 응답 정보

Entity 헤더 : 엔티티 바디 정보 예) Content-Type 등등

- 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
- 데이터 유형, 길이, 압축 정보 등등

**현재**

엔티티 → 표현

메시지 본문을 통해 표현 데이터 전달

메시지 본문 = 페이로드

표현은 요청이나 응답에서 전달 실제 데이터

표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공

- 데이터 유형, 길이, 압축 정보 등등

## **표현**

표현 헤더는 전송, 응답 둘다 사용

**Content-Type**

표현 데이터의 형식 설명

- 미디어 타입, 문자 인코딩
- text/html; charset=utf-8, application/json, image/png

**Content-Encoding**

표현 데이터 인코딩

- 표현 데이터를 압축하기 위해 사용
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
- gzip, deflate, identity(압축 안함)

**Content-Language**

표현 데이터의 자연 언어

- 표현 데이터의 자연 언어를 표현
- ko, en, en-US

**Content-Length**

표현 데이터의 길이

- 바이트 단위
- Transfer-Encoding을 사용하면 사용하면 안됨

### **협상(콘텐츠 네고시에이션)**

클라이언트가 선호하는 표현 요청

- Accept : 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset : 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding : 클라이언트가 선호하는 압축 인코딩
- Accept-Language : 클라이언트가 선호하는 자연 언어
- 협상 헤더는 요청시에만 사용

**협상과 우선순위1**

Quality Values(q)

- Quality Values(q) 값 사용
- 0 ~1, 클수록 높은 우선순위
- 생략하면 1

**협상과 우선순위2**

- 구체적인 것이 우선
- Accept : text/*, text/plain, text/plain;format=flowed

**협상과 우선순위3**

- 구체적인 것을 기준으로 미디어 타입을 맞춤

## 전송 방식

**단순 전송 :** Content-Length를 알 수 있을 때 사용

**압축 전송 :** 압축해서 전송, Content-Encoding으로 압축되어 있는 정보 전송

**분할 전송 :** Transfer-Encoding : Content-Length 보내면 안됨(예측 불가능이기 때문)

**범위 전송 :** Range, Content-Range, ex) 이미 절반 받았고 나머지 절반 달라

## **일반 정보**

**From**

유저 에이전트의 이메일 정보

- 일반적으로 잘 사용되지 않음
- 검색 엔진 같은 곳에서 주로 사용
- 요청에서 사용

**Referer**

이전 웹 페이지 주소

- 현재 요청된 페이지의 이전 웹 페이지 주소
- A -> B로 이동하는 경우 B를 요청할 때 Referer: A를 포함해서 요청
- 유입 경로 분석 가능
- 요청에서 사용

**User-Agent**

- 유저 에이전트 애플리케이션 정보(웹 브라우저 정보, 등등)
- 통계 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- 요청에서 사용

**Server**

요청을 처리하는 ORIGIN 서버의 소프트웨어 정보

- Server: Apache/2.2.22(Debian)
- server: nginx
- 응답에서 사용

**Date**

메시지가 발생한 날짜와 시간

- 응답에서 사용

## **특별한 정보**

**Host**

요청한 호스트 정보(도메인)
- 요청에서 사용
- 필수
- 하나의 서버가 여러 도메인을 처리해야 할 때
- 하나의 IP 주소에 여러 도메인이 적용되어 있을 때

**Location**

페이지 리다이렉션

- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)
- 응답코드 3xx에서 설명
- 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
- 3xx (Redirection): Location 값은 요청을 자동으로 리다이렉션하기 위한 대상 리소스를 가리킴

**Allow**

허용 가능한 HTTP 메서드

- 405 (Method Not Allowed)에서 응답에 포함해야함
- Allow: GET, HEAD, PUT

**Retry-After**

유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간

- 503: 서비스가 언제까지 불능인지 알려줄 수 있음
- 날짜 표기, 초단위 표기

## 인증

**Authorization**

클라이언트 인증 정보를 서버에 전달

**WWW-Authenticate**

리소스 접근시 필요한 인증 방법 정의

- 리소스 접근시 필요한 인증 방법 정의
- 401 응답과 함께 사용

## **쿠키**

**쿠키**
Set-Cookie : 서버에서 클라이언트로 쿠키 전달
Cookie : 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청 시 서버로 전달

**도메인**
명시한 문서 기준 도메인 + 서브 도메인 포함
domain=example.org를 지정해서 쿠키 생성
생략 시 현재 문서 기준 도메인만 적용

**경로**
경로를 포함한 하위 경로 페이지만 쿠키 접근
일반적으로 path=/ 루트로 지정

**보안**
Secure : 적용 시 https인 경우에만 전송
HttpOnly : XSS 공격 방지, HTTP 전송에만 사용
SameSite : XSRF 공격 방지, 요청 도메인과 쿠키에 설정된 메인이 같은 경우만 쿠키 전송

### 검증 헤더와 조건부 요청

캐시 유효 시간이 초과해도, 서버의 데이터가 갱신되지 않으면

304 Not Modified + 헤더 메타 정보만 응답(바디 X)

결과적으로 네트워크 다운로드가 발생하지만 용량이 적은 헤더 정보만 다운로드

**ETag**

캐시용 데이터에 임의의 고유한 버전 이름을 달아둠

데이터가 변경되면 이름을 바꾸어서 변경함

ETag만 보내서 같으면 유지, 다르면 다시 받기

If-None-Match 실패면 304 Not Modified

캐시 제어 로직을 서버에서 완전히 관리

클라이언트는 단순히 이 값을 서버에 제공

## **캐시 제어 헤더**

**Cache-Control**

캐시 지시어

max-age : 캐시 유효 시간, 초 단위

no-cache : 데이터는 캐시해도 되지만, 항상 원서버에 검증하고 사용

- 원서버 : 중간에 캐시 서버가 있고 거쳐서 원서버에 접근

no-store : 데이터에 민감한 정보가 있으므로 저장하면 안됨

**Pragma**

캐시 제어(하위 호환)

**Expires**

캐시 만료일 지정(하위 호환)

- 캐시 만료일을 정확한 날짜로 지정 (초 단위에 비해 유연하지 않음)
- max-age 권장 같이 사용하면 expires는 무시

원 서버에 바로 도달하려면 시간이 오래 걸림 -> 프록시 캐시(public) 도입

public: 응답이 public 캐시에 저장되어도 됨

private(웹 브라우저): 응답이 해당 사용자만을 위한 것임, private 캐시에 저장해야 함

s-maxage: 프록시 캐시에만 적용되는 max-age

Age: 60: 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)

**확실한 캐시 무효화 응답**

Cache-Control: no-cache, no-store, must-revalidate

Pragma: no-cache

캐시 지시어 - 확실한 캐시 무효화

**must-revalidate**

- 캐시 만료후 최초 조회시 원 서버에 검증해야함
- 원 서버 접근 실패시 반드시 오류가 발생해야함 -504

**no-cache vs must-revalidate**

1. 캐시 서버 요청 (no-cache + ETag)
2. 원 서버 요청 (no-cache + ETag)
3. 원 서버 검증
4. 원 서버 -> 프록시 캐시 응답 (304 Not Modified)
5. 캐시 -> 원 서버 응답(304 Not Modified)
6. 캐시 데이터 사용

**no-cache**

원 서버에 접근할 수 없는 경우 캐시 서버 설정에 따라서 캐시 데이터를 반환할 수 있음 -> 오류 보다는 오래된 데이터라도 보여주자

**must-revalidate**

원 서버에 접근할 수 없는 경우, 항상 오류가 발생해야 함

ex) 504 Gateway Timeout (돈과 관련된 결과일 경우)