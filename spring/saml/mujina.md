## Idp
![img.png](../../image/mujina1.PNG)
![img.png](../../image/mujina2.PNG)  
![img.png](../../image/mujina3.PNG)  

### ForceAuthnFilter
- **`forceAuthn = true`**
    - 사용자가 **IdP에서 인증된 세션**을 가지고 있어도, SAML 요청이 들어올 때마다 **무조건 재인증**을 요구한다.
    - 사용자가 **세션을 변경하거나** 브라우저를 새로 열거나 다른 세션에서 액세스할 때, 무조건 **다시 로그인**을 해야한다.
- **`forceAuthn = false`**
    - 사용자가 이미 **인증된 세션**이 있는 경우, SAML 요청이 들어와도 **기존 세션을 유지**하며, **재로그인을 요구하지 않는다**.
    - 세션이 변경되더라도, 사용자가 **여전히 유효한 세션을 보유하고 있다면** 재인증 없이 계속 인증된 상태로 유지된다.

- - -
## SP  
### WebSSOProfileConsumerImpl
* SAML Response 검증 클래스  
```java
public SAMLCredential processAuthenticationResponse(SAMLMessageContext context) throws SAMLException, SecurityException, ValidationException, DecryptionException {
    AuthnRequest request = null;
    SAMLObject message = context.getInboundSAMLMessage();
    if (!(message instanceof Response)) {
        throw new SAMLException("Message is not of a Response object type");
    } else {
        Response response = (Response)message;
        StatusCode statusCode = response.getStatus().getStatusCode();
        if (!"urn:oasis:names:tc:SAML:2.0:status:Success".equals(statusCode.getValue())) {
            StatusMessage statusMessage = response.getStatus().getStatusMessage();
            String statusMessageText = null;
            if (statusMessage != null) {
                statusMessageText = statusMessage.getMessage();
            }
      ...
}
```
