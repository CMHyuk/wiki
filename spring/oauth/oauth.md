### **Authorization Code Grant - 권한 부여 승인 코드 방식**  
권한 부여 승인을 위해 자체 생성한 Authorization Code를 전달하는 방식으로 많이 쓰이고 기본이 되는 방식이다. 간편 로그인 기능에서 사용되는 방식으로 클라이언트가 사용자를 대신하여 특정 자원에 접근을 요청할 때 사용되는 방식이다. 보통 타사의 클라이언트에게 보호된 자원을 제공하기 위한 인증에 사용된다. RefreshToken의 사용이 가능한 방식이다.  

![img.png](../../image/AuthorizationCode.jpg)  

권한 부여 승인 요청 시 response_type을 code로 지정하여 요청한다. 이후 클라이언트는 권한 서버에서 제공하는 로그인 페이지를 브라우저에 띄워 출력한다. 사용자가 로그인을 하면 권한 서버는 권한 부여 승인 코드 요청 시 전달받은 redirect_url로 Authorization Code를 전달한다. Authoirzation Code는 권한 서버에서 제공하는 API를 통해 accessToken으로 교환된다.  


### **Implicit Grant - 암묵적 승인 방식**

자격 증명을 안전하게 저장하기 힘든 클라이언트 (javaScript 등의 스크립트 언어를 사용한 브라우저)에게 최적화된 방식이다. 암시적 승인 방식에서는 권한 부여 승인 코드 없이 바로 accessToken이 발급된다. accessToken이 바로 전달되므로 만료 기간을 짧게 설정해 누출의 위험을 줄일 필요가 있다.

refreshToken 사용이 불가능한 방식이며, 이 방식에서 권한 서버는 client_secret를 사용해 클라이언트를 인증하지 않는다. accessToken을 획득하기 위한 절차가 간소화되기에 응답성과 효율성은 높아지지만 accessToken이 URL로 전달되는 단점이 있다.  

![img.png](../../image/implict.png)    

권한 부여 승인 요청 시 response_type을 token으로 설정하여 요청한다. 이후 클라이언트는 권한 서버에서 제공하는 로그인 페이지를 브라우저를 띄워 출력하게 되며 로그인이 완료되면 권한 서버는 Authorization Code가 아닌 accessToken을 redirect_url로 바로 전달한다.

### **Resource Owner Password Credentials Grant - 자원 소유자 자격 증명 승인 방식**  
간단하게 username, password로 accessToken을 받는 방식이다. 클라이언트가 타사의 외부 프로그램일 경우에 이 방식을 적용하면 안된다. 자신의 서비스에서 제공하는 어플리케이션일 경우에만 사용되는 인증 방식이다. refreshToken의 사용도 가능하다.

![img.png](../../image/resource.png)  

위와 같이 흐름은 간단하다. 제공하는 API를 통해 username, password를 전달해 accessToken을 받는 것이다. 중요한 점은 이 방식은 권한 서버, 리소스 서버, 클라이언트가 모두 같은 시스템에 속해 있을 때 사용되어야 하는 방식이라는 점이다.

### **Client Credentials Grant - 클라이언트 자격 증명 승인 방식**  
클라이언트의 자격 증명(client_id, client_secret)만으로 accessToken을 획득하는 방식이다. OAuth2의 권한 부여 방식 중 가장 간단한 방식으로 클라이언트 자신이 관리하는 리소스 혹은 권한 서버에 해당 클라이언트를 위한 제한된 리소스 접근 권한이 설정되어 있는 경우에 사용된다. 이 방식은 자격 증명을 안전하게 보관할 수 있는 클라이언트에서만 사용되어야 하며, refreshToken은 사용할 수 없다.  

![img.png](../../image/credentials.png)   


**Authorization Code 사용 유무에 따른 보안성 차이**

Authorization Code 타입에서 authorization code를 사용하면, 사용자의 브라우저와 클라이언트 사이에 존재하는 중간 공격자가 authorization code를 훔쳐가더라도 access token을 얻을 수 없으며 훔치기도 어렵다.

그 이유는 아래와 같다.

1. authorization code는 클라이언트와 인증 서버 사이에서만 사용된다.

   인증 서버는 authorization code를 발급할 때, 이를 클라이언트에게 보내지만, 사용자의**브라우저는 이 코드를 읽을 수 없다.**따라서 중간 공격자가 사용자의 브라우저를 통해 authorization code를 훔치더라도, client secret를 사용하여 인증 서버에 접근하기 때문에 이를 이용하여 access token을 얻을 수 없다.

    - **브라우저가 authorization code를 읽지 못하는 이유**
        - Authorization code는 브라우저에서 클라이언트 서버로 전달되는 동안 매우 짧은 시간 동안만 존재합니다. 이 과정에서 브라우저가 이 코드를 따로 읽거나 저장하지 않으며, 단지 쿼리 매개변수로 서버에 전달할 뿐이다.
2. authorization code는 한 번만 사용할 수 있다.

   authorization code를 이용하여 access token을 발급받으면, 이 authorization code는 폐기된다. 따라서 중간 공격자가 authorization code를 훔쳐가더라도, 이미 사용된 코드이기 때문에 access token을 얻을 수 없다.

3. 인증 서버와 클라이언트는 HTTPS를 사용하여 통신한다.

   HTTPS는 데이터 전송 과정에서 암호화를 수행하여 중간 공격자가 정보를 빼낼 수 없도록 보호한다. 따라서 사용자의 브라우저와 클라이언트 사이에서 authorization code를 훔치는 공격은 불가능하다.


반면에`access token`은 위와 마찬가지로 클라이언트에게 전달되기는 하지만 중간 공격자가 이를 가로채서 훔쳐갈 수 있다.

- 중간자 공격(man-in-the-middle attack) 등의 방법을 통해 훔칠 수 있다.
- access token은 브라우저에서 읽을 수 있다. (즉, 훔치면 써먹을 수 있다.)