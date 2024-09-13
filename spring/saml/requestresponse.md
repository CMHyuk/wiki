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
* SAML 프로토콜에 따라 SP가 IdP로 보내는 인증 요청이다.
   * AssertionConsumerServiceURL: SAML Response를 받을 SP의 엔드포인트 URL
   * Destination: SAML 요청을 처리할 IdP의 SSO URL
   * ProtocolBinding SAML 응답을 받을 방식, 여기서는 HTTP-POST 방식이 사용
   * ID: 요청의 고유 식별자
   * IssueInstant: 요청이 생성된 시각
     
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
* 요청 결과, urn:oasis:names:tc:SAML:2.0:status:Success는 성공을 의미

**Assertion**
* 사용자의 인증 정보를 포함한 데이터

* Issuer - Assertion을 발행한 IdP의 식별자  
* Subject - 인증된 사용자, 이 예시에서는 이메일 주소(user@example.com)로 사용자 식별자를 제공  
* SubjectConfirmation - Assertion이 특정 SP에만 사용되도록 제한하는 정보, NotOnOrAfter는 유효기간을 나타냄  
* Conditions - Assertion이 유효한 시간 범위와, Assertion이 유효한 대상(SP)을 명시  
* AudienceRestriction - 이 Assertion이 유효한 SP를 명시, 여기서는 https://sp.example.com이 유효한 SP로 설정
* AuthnStatement - 사용자가 언제 인증되었는지, 어떤 인증 방법(여기서는 PasswordProtectedTransport)을 사용했는지를 나타냄
