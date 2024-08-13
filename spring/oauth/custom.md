### 동의 화면
동의 화면 커스텀은 OAuth2AuthorizationConsentService implements 후 OAuth2AuthorizationConsent 을 매핑해서 저장

- - -

### OAuth2Authorization
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

### code, state 값 따로 저장 이유
authorizationService.findByToken(authorizationConsentAuthentication.getState(), STATE_TOKEN_TYPE); 처럼 조회한 후 로직 처리하기 때문에 OAuth2Authorization 에 담겨져 있지만 따로 필드 생성

- - -

### 로그인 고려 사항

code 생성, 저장 호출 클래스

`OAuth2AuthorizationCodeRequestAuthenticationProvider` - 동의 항목 체크 후 code 생성되는 클래스
`OAuth2AuthorizationConsentAuthenticationProvider` - 동의 항목 모두 체크된 상태에서 재로그인 시 code 생성 되는 클래스

``` java
if (this.authorizationConsentRequired.test(authenticationContextBuilder.build())) {
    String state = DEFAULT_STATE_GENERATOR.generateKey();
    OAuth2Authorization authorization = authorizationBuilder(registeredClient, principal, authorizationRequest).attribute("state", state).build();
    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Generated authorization consent state");
    }
        this.authorizationService.save(authorization);
        Set<String> currentAuthorizedScopes = currentAuthorizationConsent != null ? currentAuthorizationConsent.getScopes() : null;
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Saved authorization");
        }
        return new OAuth2AuthorizationConsentAuthenticationToken(authorizationRequest.getAuthorizationUri(), registeredClient.getClientId(), principal, state, currentAuthorizedScopes, (Map)null);
}
```
* 현재 인증된 사용자에게 특정 클라이언트(애플리케이션) 및 요청된 범위(scopes)에 대해 동의(consent)가 필요한지 여부를 검사
    * 스코프 모두 체크되면 이 로직을 타지 않고 스코프를 모두 체크하지 않은 경우 이 로직을 탐     

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

``` java
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

``` java
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

``` java
SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
                context.setAuthentication(authenticationResult);
                this.securityContextHolderStrategy.setContext(context);
                this.securityContextRepository.saveContext(context, request, response);
```

- SecurityContext에 담아 반환
    - `SecurityContextHolder.*getContext*().getAuthentication();` 로 유저 정보를 가져올 수 있음
        - ArgumentResovler와 활용 가능

- - -
### 멀티 테넌트 구조의 경우의 JWKSource

멀티 테넌트 구조의 경우 테넌트마다 public, private key가 다르기 때문에 런타임 중 각 테넌트에 맞는 키를 사용할 수 있어야 함

```java
@Bean
    public JWKSource<SecurityContext> jwkSource() throws Exception {
        KeyResponse key = masterTenantInfoQueryService.getKey("master");
        byte[] publicKeyBytes = key.pubKey();
        byte[] privateKeyBytes = key.priKey();

        RSAPublicKey rsaPublicKey = loadPublicKey(publicKeyBytes);
        RSAPrivateKey rsaPrivateKey = loadPrivateKey(privateKeyBytes);

        RSAKey rsaKey = new RSAKey.Builder(rsaPublicKey)
                .privateKey(rsaPrivateKey)
                .keyID(UUID.randomUUID().toString()) // 키 ID를 생성
                .build();

        JWKSet jwkSet = new JWKSet(rsaKey);
        return (jwkSelector, context) -> jwkSelector.select(jwkSet);
    }
```

위처럼 서버를 기동시킬 때 등록 시키면 런타임 중 새로 생성된 테넌트가 있을 경우 그 테넌트의 키는 활용하지 못함

#### JWKSource 동적으로 사용하기

`JwtEncoder`를 상속해 `CustomJwtEncoder`를 사용하도록 하기

#### 내부 로직 흐름

```java
public Jwt encode(JwtEncoderParameters parameters) throws JwtEncodingException {
    RSAKey rsaKey = getRsaKey();
    JWKSet jwkSet = new JWKSet(rsaKey);
    JWKSource<SecurityContext> jwkSource = (jwkSelector, context) -> jwkSelector.select(jwkSet);

    JwsHeader headers = parameters.getJwsHeader();
    JwtClaimsSet claims = parameters.getClaims();
    JWK jwk = this.selectJwk(jwkSource, headers);
    headers = addKeyIdentifierHeadersIfNecessary(headers, jwk);
    String jws = this.serialize(headers, claims, jwk);
    return new Jwt(jws, claims.getIssuedAt(), claims.getExpiresAt(), headers.getHeaders(), claims.getClaims());
}

private RSAKey getRsaKey() {
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    String clientId = authentication.getName();

    ClientInfo clientInfo = clientInfoQueryRepository.findByClientId(clientId)
            .orElseThrow(() -> BusinessException.from(UserErrorCode.NOT_FOUND));
    TenantInfo tenantInfo = tenantInfoQueryRepository.findByTenantId(clientInfo.getTenantId())
            .orElseThrow(() -> BusinessException.from(TenantErrorCode.NOT_FOUND));

    byte[] publicKeyBytes = tenantInfo.getTenantRSAPublicKey();
    byte[] privateKeyBytes = tenantInfo.getTenantRSAPrivateKey();

    RSAPublicKey rsaPublicKey = loadPublicKey(publicKeyBytes);
    RSAPrivateKey rsaPrivateKey = loadPrivateKey(privateKeyBytes);

    return new RSAKey.Builder(rsaPublicKey)
            .privateKey(rsaPrivateKey)
            .keyID(UUID.randomUUID().toString())
            .build();
}
```

현재 접근한 자원이 누구인지 추출 후 db에 저장된 public, private key로 jwkSource 등록 후 이를 활용해 encode 하도록 구현

- - -

### 토큰 Decode 커스텀

encode와 마찬가지로 decode 시 테넌트마다 공개키가 다르므로 이에 맞는 공개키를 사용해야함

```java
@Bean
public JwtDecoder jwtDecoder() throws Exception {
      KeyResponse key = tenantInfoService.getKey("oauth-client-id");
      RSAPublicKey rsaPublicKey = loadPublicKey(key.pubKey());
      return NimbusJwtDecoder.withPublicKey(rsaPublicKey).build();
}    
```

위처럼 하면 동적으로 키를 사용할 수 없음, 하지만 처음 서버를 기동할 때 무조건 하나는 등록시켜야 에러가 나지 않기 때문에 마스터 테넌트에 대한 공개키만 등록

**문제**

공개키를 동적으로 사용하기 위해  `NimbusJwtDecoder` 대신 커스텀 하면 좋지만 구현하기 까다로움 (공개키를 가져오기 위한 서비스 클래스 의존 관계 주입으로 코드가 꼬임)

**해결**  
`JwtAuthenticationProvider` 대신 커스텀해서 decode전 넘어온 토큰 먼저 파싱하고 담긴 정보로 공개키를 가져와서 등록시킨 후 decode 하도록 구현

**단점**
- 로직의 중복
    - `JWT parse = JWTParser.*parse*(token);`  decode 클래스에서 호출하는데 공개 키를 가져오기 위해 어쩔 수 없이 `CustomAuthenticationProvider`에서도 호출
      - `JwtDecoder` 대신 커스텀 방법 더 고민 필요