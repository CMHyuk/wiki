![img.png](../../image/samlflow.png)

**GET /private**  
사용자가 브라우저에서 보호된 리소스에 접근하기 위해 GET 요청을 보냅니다.  

**AccessDeniedException 발생**  
해당 리소스는 인증된 사용자만 접근 가능하기 때문에, AccessDeniedException이 발생합니다. 이때 LoginUrlAuthenticationEntryPoint가 활성화됩니다.  

**Location 리디렉션: /saml2/authenticate/example**  
브라우저는 /saml2/authenticate/example URL로 리디렉션됩니다.  

**GET /saml2/authenticate/example**  
브라우저가 리디렉션된 후 /saml2/authenticate/example에 GET 요청을 보내고, 이는 IDP(Idenitity Provider, 인증 제공자)로 리디렉션됩니다.  

**IDP로 리디렉션 및 AuthnRequest 전송**  
브라우저는 IDP (https://idp.example.com)로 GET 요청을 보내고, 이 과정에서 AuthnRequest가 IDP에 전송됩니다.  

**POST /login/saml2/sso/example**  
IDP가 사용자 인증을 완료한 후, 응답을 SP(Service Provider)에 전달하기 위해 브라우저는 /login/saml2/sso/example로 POST 요청을 보냅니다.