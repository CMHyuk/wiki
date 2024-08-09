#### 동의 화면
동의 화면 커스텀은 OAuth2AuthorizationConsentService implements 후 OAuth2AuthorizationConsent 을 매핑해서 저장

- - -

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

- - -

#### code, state 값 따로 저장 이유
authorizationService.findByToken(authorizationConsentAuthentication.getState(), STATE_TOKEN_TYPE); 처럼 조회한 후 로직 처리하기 때문에 OAuth2Authorization 에 담겨져 있지만 따로 필드 생성

- - -

**로그인 고려 사항**

code 생성, 저장 호출 클래스

`OAuth2AuthorizationCodeRequestAuthenticationProvider` - 동의 항목 체크 후 code 생성되는 클래스
`OAuth2AuthorizationConsentAuthenticationProvider` - 동의 항목 모두 체크된 상태에서 재로그인 시 code 생성 되는 클래스

1. 최초 로그인 시 `currentAuthorizationConsent`은 null이므로 state만 생성해서 `OAuth2Authorization` 저장하고 동의 화면에서 스코프 체크 후 code 생성
    1.  재 로그인 시 `currentAuthorizationConsent`은 null이 아니어서 state는 생성하지 않고, 최초 로그인 시 동의 항목 모두 체크했으면 바로 code 생성하고 `save()` 호출, 모두 체크하지 않았다면 최초 로그인과 같은 방식으로 code 생성 후 `save()`

그래서 두 가지 상황을 고려해 시큐리티에서 넘겨준 `authorization`의`OAuth2Authorization.Token<OAuth2AuthorizationCode>`가 null이면 code 값은 비워두고 존재하면 값 꺼내서 저장하도록 구현 (그냥 넘어온 값 저장하게 하면 NPE 발생)

- - -

**accessToken 저장 고려 사항**

토큰 생성, 저장 호출 클래스
`OAuth2AuthorizationCodeAuthenticationProvider` 

1. 로그인, 동의 화면에 거쳐서 만들어진 code로 `OAuth2Authorization` 가져옴
2. code가 이미 사용된 코드인지, 기간 만료됐는지 확인
3. accessToken 생성 후 `registeredClient`가 가지고 있는 `AuthorizationGrantType`을 체크하며 존재하면 refreshToken, idToken 생성
4. code는 사용했으므로 메타데이터 값 false에서 true로 바꾼 후 `save()` 호출

토큰 저장 시 만료 된 코드로 저장되지 않게 하려면 db에 저장된 `OAuth2Authorization` 의 code 메타데이터 값을 확인하고 false면 저장하고 true면 저장 안 되도록 한 후 시큐리티에서 넘겨준 `OAuth2Authorization`를 db에 업데이트

**토큰 생성도 커스텀 한다면 아래 참고**

https://garnier.wf/blog/2024/02/12/spring-auth-server-tokens.html

- - -

### **토큰 검증 흐름**

`NimbusJwtDecoder` 클래스

```
public Jwt decode(String token) throws JwtException {
        JWT jwt = this.parse(token);
        if (jwt instanceof PlainJWT) {
            this.logger.trace("Failed to decode unsigned token");
            throw new BadJwtException("Unsupported algorithm of " + jwt.getHeader().getAlgorithm());
        } else {
            Jwt createdJwt = this.createJwt(token, jwt);
            return this.validateJwt(createdJwt);
        }
    }
```

- 토큰 파싱 → Jwt 생성 후 검증
- `PlainJWT`는 서명이 없는 JWT  - `alg`(알고리즘) 헤더가 `"none"`으로 설정되며, 이는 토큰이 서명되지 않았음을 의미

**검증**

`DelegatingOAuth2TokenValidator`에서 `JwtTimestampValidator` 호출

```
public OAuth2TokenValidatorResult validate(Jwt jwt) {
        Assert.notNull(jwt, "jwt cannot be null");
        Instant expiry = jwt.getExpiresAt();
        if (expiry != null && Instant.now(this.clock).minus(this.clockSkew).isAfter(expiry)) {
            OAuth2Error oAuth2Error = this.createOAuth2Error(String.format("Jwt expired at %s", jwt.getExpiresAt()));
            return OAuth2TokenValidatorResult.failure(new OAuth2Error[]{oAuth2Error});
        } else {
            Instant notBefore = jwt.getNotBefore();
            if (notBefore != null && Instant.now(this.clock).plus(this.clockSkew).isBefore(notBefore)) {
                OAuth2Error oAuth2Error = this.createOAuth2Error(String.format("Jwt used before %s", jwt.getNotBefore()));
                return OAuth2TokenValidatorResult.failure(new OAuth2Error[]{oAuth2Error});
            } else {
                return OAuth2TokenValidatorResult.success();
            }
        }
    }
```

- null 인지, 만료된 토큰인지 체크

이상 없으면 `BearerTokenAuthenticationFilter` 에서

```
SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
                context.setAuthentication(authenticationResult);
                this.securityContextHolderStrategy.setContext(context);
                this.securityContextRepository.saveContext(context, request, response);
```

- SecurityContext에 담아 반환
    - `SecurityContextHolder.*getContext*().getAuthentication();` 로 유저 정보를 가져올 수 있음
        - ArgumentResovler와 활용 가능
