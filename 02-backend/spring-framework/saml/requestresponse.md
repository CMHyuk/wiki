### SAML Request (Authentication Request)  
SP가 IdP로 인증 요청을 보낼 때 사용되는 XML 기반 메시지

``` xml
<samlp:AuthnRequest 
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" 
    AssertionConsumerServiceURL="https://sp.example.com/acs"
    Destination="https://idp.example.com/sso"
    ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" 
    ID="_1234567890abcdef" 
    Version="2.0" 
    IssueInstant="2024-09-13T12:00:00Z">
  
    <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
        https://sp.example.com
    </saml:Issuer>
  
    <samlp:NameIDPolicy 
        Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress" 
        AllowCreate="true"/>
  
    <samlp:RequestedAuthnContext Comparison="exact">
        <saml:AuthnContextClassRef>
            urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
        </saml:AuthnContextClassRef>
    </samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

**AuthnRequest**  
* SAML 프로토콜에 따라 SP가 IdP로 보내는 인증 요청
   * AssertionConsumerServiceURL - SAML Response를 받을 SP의 엔드포인트 URL
   * Destination - SAML 요청을 처리할 IdP의 SSO URL
   * ProtocolBinding SAML - 응답을 받을 방식, 여기서는 HTTP-POST 방식이 사용
   * ID - 요청의 고유 식별자
   * IssueInstant - 요청이 생성된 시각
     
**Issuer**
* SP의 Entity ID로, SP를 식별하는 URL
  
**NameIDPolicy**
* IdP가 사용자 식별자를 어떤 형식으로 전달해야 하는지 명시, 위 예시에서는 이메일 주소 형식`(urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress)`을 요청
  
**RequestedAuthnContext**  
  * SP가 요구하는 인증 강도를 명시, 위 예시에서는 PasswordProtectedTransport(비밀번호 보호된 전송) 방식의 인증을 요구

- - -
###  SAML Response (Authentication Response)
IdP가 SP로 사용자의 인증 정보를 전달할 때 사용하는 XML 메시지  

``` xml
<samlp:Response 
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
    Destination="https://sp.example.com/acs"
    ID="_abcdef1234567890"
    InResponseTo="_1234567890abcdef"
    IssueInstant="2024-09-13T12:01:00Z"
    Version="2.0">
  
    <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
        https://idp.example.com
    </saml:Issuer>
  
    <samlp:Status>
        <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
    </samlp:Status>
  
    <saml:Assertion xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" 
                    ID="_a1b2c3d4e5f6g7h8"
                    IssueInstant="2024-09-13T12:01:00Z"
                    Version="2.0">
    
        <saml:Issuer>https://idp.example.com</saml:Issuer>
    
        <saml:Subject>
            <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
                user@example.com
            </saml:NameID>
            <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <saml:SubjectConfirmationData 
                    InResponseTo="_1234567890abcdef"
                    NotOnOrAfter="2024-09-13T12:06:00Z"
                    Recipient="https://sp.example.com/acs"/>
            </saml:SubjectConfirmation>
        </saml:Subject>
    
        <saml:Conditions NotBefore="2024-09-13T12:00:00Z" NotOnOrAfter="2024-09-13T12:10:00Z">
            <saml:AudienceRestriction>
                <saml:Audience>https://sp.example.com</saml:Audience>
            </saml:AudienceRestriction>
        </saml:Conditions>
    
        <saml:AuthnStatement AuthnInstant="2024-09-13T12:00:30Z" SessionIndex="_a1b2c3d4">
            <saml:AuthnContext>
                <saml:AuthnContextClassRef>
                    urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
                </saml:AuthnContextClassRef>
            </saml:AuthnContext>
        </saml:AuthnStatement>
    
    </saml:Assertion>
</samlp:Response>
```
**Response**  
* Destination - SP의 Assertion Consumer Service (ACS) URL로, SAML 응답을 받는 엔드포인트  
* ID - 응답의 고유 식별자  
* InResponseTo - SP가 보낸 요청의 ID로, 이 응답이 어느 요청에 대한 것인지 명시

**Issuer**  
* 이 응답을 발행한 IdP의 Entity ID로, IdP를 식별하는 URL

**Status**  
* 요청 결과, `urn:oasis:names:tc:SAML:2.0:status:Success`는 성공을 의미

**Assertion**
* Issuer - Assertion을 발행한 IdP의 식별자  
* Subject - 인증된 사용자, 이 예시에서는 이메일 주소(user@example.com)로 사용자 식별자를 제공  
* SubjectConfirmation - Assertion이 특정 SP에만 사용되도록 제한하는 정보, NotOnOrAfter는 유효기간을 나타냄  
* Conditions - Assertion이 유효한 시간 범위와, Assertion이 유효한 대상(SP)을 명시  
* AudienceRestriction - 이 Assertion이 유효한 SP를 명시, 여기서는 https://sp.example.com이 유효한 SP로 설정
* AuthnStatement - 사용자가 언제 인증되었는지, 어떤 인증 방법(여기서는 PasswordProtectedTransport)을 사용했는지를 나타냄

- - -
### SAML Response 검증
1. **SAML Response 수신**  
   사용자가 인증을 마치고 나면, IdP는 SAML Response를 사용자의 브라우저를 통해 SP의 ACS URL로 전송
   이 응답은 보통 HTTP POST 방식으로 전송되며, SAML Response는 XML 형식으로 Base64로 인코딩되어 전달
2. **SAML Response 디코딩**  
   SP는 HTTP POST 요청으로 전달된 SAML Response를 Base64 디코딩하여 원래의 XML 형태로 변환
   이 XML은 SAML 표준에 맞춰 구성되어 있으며, 인증 관련 정보(사용자의 ID, 인증 세부 사항 등)가 포함됨
3. **SAML 서명 검증 (Signature Verification)**  
   SP는 이 서명이 IdP의 공개 키로 유효한지 검증을 통해 SAML Response가 IdP에서 온 것이며, 중간에 변조되지 않았음을 확인
   이 과정에서 SP는 IdP의 인증서(공개 키)를 사용해 서명을 검증, 이 인증서는 보통 SAML 메타데이터를 통해 사전에 교환
   서명 검증 실패 시: 만약 서명이 유효하지 않다면, SAML Response는 무효화되고 사용자 인증은 실패로 처리
4. **SAML Assertion 검증**  
   SAML Response 내부에는 SAML Assertion이 포함
   Assertion의 중요한 필드들을 검증하는 과정이 포함 
   * IssueInstant: Assertion이 발행된 시간, 이 값이 너무 오래된 경우(예: 발행된 지 오래되거나, 만료 시간이 지난 경우) 요청이 무효화
   * Conditions: Assertion에 적용되는 조건, 예를 들어 Assertion이 유효한 시간 범위(NotBefore, NotOnOrAfter 필드)나 특정 대상(SP)에게만 유효하도록 제한된 경우
   * AudienceRestriction: Assertion이 특정 SP에만 유효하도록 제한, 이 필드는 SAML Assertion이 이 SP를 위한 것인지 확인하는 데 사용 
   * Destination: SAML Response가 도착한 목적지(즉, ACS URL)가 Assertion의 Destination 필드에 명시된 값과 일치하는지 검증
5. **사용자 정보 추출 및 검증**  
   Assertion 내에서 사용자 정보(Subject)를 추출, 이 정보에는 사용자 ID, 이메일 또는 기타 인증된 정보가 포함
   SP는 이 사용자 정보가 SP 시스템에서 유효한지 추가로 검증 가능
   예를 들어, 해당 사용자 ID가 SP 시스템에 존재하는지, 추가적인 권한 또는 접근 제어가 필요한지 확인 가능
6. **세션 생성**  
   SAML Assertion이 검증되면, SP는 사용자에게 세션을 생성하고 사용자를 서비스에 로그인 상태로 유지.
   이후 사용자는 별도의 로그인 절차 없이 SP의 서비스를 사용 가능
7. **검증 실패 시 처리**  
   만약 검증 과정에서 문제가 발생하면(예: 서명이 유효하지 않거나, Assertion이 만료되었거나, Audience가 일치하지 않는 경우), SP는 인증 요청을 거부하고 사용자를 로그인 페이지로 리다이렉트하거나 오류 메시지를 표시

**SAML Response 검증 요약**  
* Base64 디코딩: SAML Response를 Base64에서 디코딩하여 XML로 변환
* 전자 서명 검증: SAML Response가 IdP에서 발행되었고 변조되지 않았는지 확인
* SAML Assertion 검증
  * 발행 시간(IssueInstant)과 만료 시간(NotOnOrAfter) 확인 
  * Audience(SP)가 올바른지 확인
  * Destination이 ACS URL과 일치하는지 확인
  * 사용자 정보 확인: Assertion 내의 사용자 정보 추출 및 검증 
  * 세션 생성: SAML Assertion이 검증되면 사용자에게 세션 생성