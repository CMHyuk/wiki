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

- - -
### 한 번도 로그인하지 않은 경우
1. **사용자가 SP에 접근**  
   사용자가 웹 브라우저를 통해 서비스 제공자(SP)에 접근한다. 사용자는 아직 인증되지 않은 상태이다.
2. **SP에서 세션 확인**  
   SP는 사용자가 로그인했는지 확인하기 위해 세션 정보를 확인한다. 세션이 없다면, SP는 사용자를 인증할 수 없으므로 IdP로 리다이렉트해야 한다.
3. **SP에서 IdP로 리다이렉트**  
   SP는 사용자를 인증하기 위해 SAML 인증 요청을 IdP로 리다이렉트한다.
   이때 SP는 SAML AuthnRequest라는 메시지를 IdP로 전송한다. 이 메시지에는 SP 정보, 사용자 식별 정보가 포함되며, 이 정보는 SAML 규격에 맞춰 XML 형식으로 암호화된다.
4. **IdP에서 로그인 페이지로 이동**  
   IdP는 SP로부터 받은 SAML AuthnRequest를 처리한 후, 사용자가 이미 로그인했는지 확인한다. 이 시점에서 사용자가 로그인된 상태가 아니라면, IdP는 사용자에게 로그인 페이지를 제공한다.
5. **사용자 로그인 및 IdP에서 세션 저장**  
   사용자는 IdP의 로그인 페이지에서 자격 증명(예: 사용자명, 비밀번호)을 입력하고 인증에 성공한다.
   IdP는 사용자의 세션 정보를 생성하고 저장한다.
6. **IdP에서 SAML 응답 생성 및 SP로 리다이렉트**  
   사용자가 성공적으로 로그인하면, IdP는 SAML Assertion이라는 인증 응답을 생성하여 SP로 리다이렉트한다.
   이 SAML Assertion에는 사용자가 인증되었다는 정보를 포함하며, 일반적으로 서명되고 암호화되어 전송된다.
7. **SP에서 세션 저장 및 사용자 서비스 접근 허용**  
   SP는 IdP로부터 받은 SAML Assertion을 검증한 후, 사용자 세션을 생성하고 저장한다.
   이후 사용자는 SP에서 제공하는 서비스를 사용할 수 있게 된다.

- - -

### 기존에 한 번 이상 로그인한 경우
1. **사용자가 SP에 접근**  
   사용자가 SP에 다시 접근한다. 이때는 이미 이전에 IdP에서 인증된 상태이다.
2. **SP에서 세션 확인**  
   SP는 사용자의 세션 정보를 확인한다. 만약 사용자가 아직 SP에 세션이 없다면, IdP로 인증 요청을 다시 리다이렉트해야 한다.
3. **SP에서 IdP로 리다이렉트**  
   SP에 세션이 없으면, 다시 SAML AuthnRequest를 IdP로 전송하여 사용자가 인증되어 있는지 확인한다.
   만약 SP에 세션이 있는 경우, 바로 서비스에 접근하게 된다.
4. **IdP에서 세션 확인**  
   IdP는 사용자가 이전에 인증한 세션이 남아 있는지 확인한다. 이미 세션이 존재한다면, 사용자는 로그인 절차 없이 바로 인증이 완료된 것으로 간주된다.
   IdP는 로그인 페이지를 보여주지 않고, SAML Assertion을 다시 생성하여 SP로 리다이렉트한다.
5. **SP에서 세션 저장 및 사용자 서비스 접근 허용**  
   SP는 IdP에서 전달받은 SAML Assertion을 검증하고, 사용자에게 새로운 세션을 생성한다.
   이후 사용자는 SP에서 제공하는 서비스를 사용할 수 있다.