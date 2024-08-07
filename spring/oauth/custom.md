#### 동의 화면
동의 화면도 커스텀은 OAuth2AuthorizationConsentService implements 후 OAuth2AuthorizationConsent 을 매핑해서 저장

#### OAuth2Authorization
**`OAuth2Authorization` 저장 과정**

- `save()` 가 총 3번이 호출
    1. **로그인** - code는 생성하지 않고 (state는 생성) 그냥 저장
    2. **동의 항목 체크** - code를 생성하고 state는 삭제한 후 `OAuth2Authorization` 에 code 담아서 `save()`
        1. **code 생성 과정**
            1.  `OAuth2AuthorizationConsentAuthenticationProvider`에서
                `(OAuth2AuthorizationCode)this.authorizationCodeGenerator.generate(tokenContext);`로 생성하고 `OAuth2Authorization`에 담아 다시 `save()` 호출한 후 리다이렉트
    3. **accessToken** **발급 요청** - 토큰을 생성해서 `OAuth2Authorization`에 담아 `save()` 호출

위 과정을 고려해 `save()` 커스텀 로직 작성

#### code, state 값 따로 저장 이유
authorizationService.findByToken(authorizationConsentAuthentication.getState(), STATE_TOKEN_TYPE); 처럼 조회한 후 로직 처리하기 때문에 OAuth2Authorization 에 담겨져 있지만 따로 필드 생성