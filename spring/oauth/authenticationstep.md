
![img.png](../../image/login.png)

1. `AuthenticationConverter` 에서 정확히 필요한 값들이 왔는지 확인 (redirect_uri, client_id 등등..) 이상이 없으면 `additionalParameteres` 에 값을 넣어서 인증 객체 생성 및 저장
    1. `state`는 `CSRF`를 위해 사용이 권장된다.
    2. `PKCE` 더 강력한 보안을 위해 사용
        1. 공개 클라이언트의 경우 필수이지만 기밀 클라이언트는 필수가 아님
            1. 공개 vs 기밀의 차이? → 일반적인 경우는 공개, 사내에서만 쓰면 기밀
2. `Provider`가 인증을 하는 과정은  `registeredClient`(1번에서 저장한 값)를 가져와서 request에서 넘어온 값들이 같은지 비교하면서 이루어진다.
3. 해당 항목들이 일치하는지 확인하고, 현재 `AuthenticationToken`의 `principal` 은 `AnonymousUser` 즉, 인증이 안 되었기 때문에 `Resource server`에 `user`가 인증을 할 수 있도록 로그인 페이지로 넘긴다.