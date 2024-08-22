### OAuth2.1에서 달라진 점
1. PKCE : 모든 OAuth 클라이언트가 Authorization Code Grant flow를 사용할 때 PKCE(Proof Key for Code Exchange)를 필수적으로 사용해야 한다. 이는 코드 교환 과정의 보안을 강화하기 위함이다.
2. 리다이렉트 URI의 정확한 문자열 일치 비교: 리다이렉트 URI는 정확한 문자열 매칭을 사용하여 비교해야 합니다. 이는 보안을 강화하기 위한 조치이다.
3. Implicit, Resource Owner Password Credentials 제외: OAuth 2.1 사양에서 제외되었습니다. 이는 보안 취약점을 줄이기 위한 조치이다.
4. Bearer Token 사용의 제한: URI의 쿼리 문자열에서 베어러 토큰(bearer tokens) 사용을 금지합니다. 이는 보안 위험을 감소시키기 위함이다.
5. 공개 클라이언트의 Refresh token 제한: 공개 클라이언트(public clients)에 대한 리프레시 토큰은 발신자 제약(sender-constrained)이 있거나 일회용(one-time use)이어야 한다.
6. 공개 및 기밀 클라이언트의 정의 간소화: 공개 및 기밀 클라이언트의 정의가 클라이언트에 자격 증명이 있는지 여부로만 표시되도록 간소화되었다.

### PKCE 란?  
Proof Key for Code Exchange 의 약어로써 Authorization Code Grant Type의 확장 개념이다.  

### PKCE에서 추가되는 필드
* code_verifier: 인증 코드(code)를 가로채지 못하도록 하는 임의의 Random key이다. 
* code_challenge: code_verifier 값을 code_challenge_method 로 Hashing 한 값이다. 
* code_challenge_method: code_challenge를 어떤 방식으로 변환할 것인지를 지정한다.  

![img.png](../../image/pkce.webp) 

1. 유저가 서비스 사용을 위해 Client application에 접근한다. 
2. Client application 에서 인증이 필요함을 인지하고 PKCE 수행하기 전 code_verifier, code_challenge 값을 생성한다.
3. Authorization Server 측으로 code 인증 방식 요청과 challenge 값을 보낸다.
4. 유저에게 Oauth 인증화면을 보여주고 인증을 시도한다.
5. redirect_uri 에 code값을 담아서 전달한다. 
6. 전달받은 code 값과 code_verifier 를 통해 token 교환을 요청한다.
7. 전달받은 값을 통해서 Authorization Server 측에서 code_verifier를 사용하여 code_challenge를 다시 계산하고, 이것이 클라이언트가 처음에 보낸 code_challenge와 일치하는지 확인한다. 
8. 만약 일치하면, 서버는 클라이언트에게 accessToken을 발급한다.  


### PKCE가 공격을 방지할 수 있는 이유는?
1. code_challenge 전송: 클라이언트는 인증 요청(/authorize)을 보낼 때 code_challenge를 인증 서버에 전송한다. 이 단계에서 code_challenge는 공개될 수 있으며, 이것 자체로는 보안 위험을 초래하지 않는다.
2. Callback code 시 : 사용자가 인증을 완료하면, Authorization Server는 클라이언트에게 code를 반환한다. 이 코드는 일반적으로 리다이렉션을 통해 클라이언트에게 전달되며, 이 과정에서 공격자가 인증 코드를 가로챌 가능성이 있다.
3. code_verifier의 역할: 공격자가 code를 가로챘다고 해도, code_verifier가 없다면 액세스 토큰을 얻을 수 없다. code_verifier는 클라이언트가 보유하고 있으며, /token 요청 시에만 인증 서버에 전송된다. 이 단계에서 code_verifier는 암호화되거나 보호되는 채널을 통해 전송되므로, 공격자가 이를 가로채기는 매우 어렵다.

**결론적으로, 공격자가 code_challenge를 가로챈다 해도, 이 정보만으로는 액세스 토큰을 획득할 수 없다.**