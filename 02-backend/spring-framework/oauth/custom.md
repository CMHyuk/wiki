### 동의 화면
동의 화면 커스텀은 OAuth2AuthorizationConsentService implements 후 OAuth2AuthorizationConsent 을 매핑해서 저장

- - -

### OAuth2Authorization
**`OAuth2Authorization` 저장 과정**

`CustomOAuth2AuthorizationService` 의 `save()` 메서드

- `save()` 호출 시점
    1. **최초 로그인**- `currentAuthorizationConsent`은 null이므로 state만 생성해`OAuth2Authorization`저장
    2. **동의 항목 체크**- code를 생성하고 state는 삭제한 후`OAuth2Authorization`에 code를 담아`save()`
        1. **code 생성 과정**
            1. `OAuth2AuthorizationConsentAuthenticationProvider`에서`(OAuth2AuthorizationCode)this.authorizationCodeGenerator.generate(tokenContext);`로 생성하고`OAuth2Authorization`에 담아 다시`save()`호출
    3. **accessToken, refreshToken 발급**- 토큰을 생성해서`OAuth2Authorization`에 담아`save()`호출
    4. **재 로그인** -`currentAuthorizationConsent`은 null이 아니어서 state는 생성하지 않고, 최초 로그인 시 동의 항목 모두 체크했으면 바로 code 생성하고`save()`호출, 모두 체크하지 않았다면 최초 로그인과 같은 방식으로 code 생성 후`save()`
    5. **토큰 재발급** - 토큰 재생성 후 `save()`

위 과정을 고려해 `save()` 커스텀 로직 작성

- - -
`OAuth2AuthorizationConsentAuthenticationProvider` 클래스  
동의 체크 아무 것도 안 하면 OAuth2AuthorizationService remove 호출
```java
if (authorities.isEmpty()) {
    if (currentAuthorizationConsent != null) {
        this.authorizationConsentService.remove(currentAuthorizationConsent);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Revoked authorization consent");
        }
     }
         this.authorizationService.remove(authorization);
         if (this.logger.isTraceEnabled()) {
             this.logger.trace("Removed authorization");
         }
         throwError("access_denied", "client_id", authorizationConsentAuthentication, registeredClient, authorizationRequest);
 }
```
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

### accessToken 저장 고려 사항

**토큰 생성, 저장 호출 클래스**
`OAuth2AuthorizationCodeAuthenticationProvider` 

#### 토큰 최초 생성
1. 로그인, 동의 화면에 거쳐서 만들어진 code로 `OAuth2Authorization` 가져옴
2. code가 이미 사용된 코드인지, 기간 만료됐는지 확인
3. accessToken 생성 후 `registeredClient`가 가지고 있는 `AuthorizationGrantType`을 체크하며 존재하면 refreshToken, idToken 생성
4. code는 사용했으므로 메타데이터 값 false에서 true로 바꾼 후 `save()` 호출

토큰 저장 시 만료 된 코드로 저장되지 않게 하려면 db에 저장된 `OAuth2Authorization` 의 code 메타데이터 값을 확인하고 false면 저장하고 true면 저장 안 되도록 한 후 시큐리티에서 넘겨준 `OAuth2Authorization`를 db에 업데이트

#### 토큰 재발급
1. `refreshToken.isActive()` 확인
1. `OAuth2Authorization`에 담긴 refreshToken으로 db에 있는지 확인
2. 존재하면 새로 발급된 토큰 저장

**토큰 재발급 참고**  

code는 한 번밖에 사용하지 못하는데, 만약 이미 사용한 code로 토큰 발급을 요청하면 accessToken, refreshToken 모두 invalidated = true로 바꿈
따라서 기존 발급된 refreshToken으로 acccessToken 재발급 불가능

 
로그인 시 토큰 무제한 발급이 아닌 정해진 횟수까지 발급 받을 수 있도록 로직 구현

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
### 멀티 테넌트 구조의 JWKSource

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

- - -

### Access Token 재발급 고려사항

**동작 흐름**  
`OAuth2RefreshTokenAuthenticationProvider` 클래스에서 refreshToken 조회 후 검증하고 재발급  
```java
OAuth2Authorization authorization = this.authorizationService.findByToken(refreshTokenAuthentication.getRefreshToken(), OAuth2TokenType.REFRESH_TOKEN);
OAuth2Authorization.Token<OAuth2RefreshToken> refreshToken = authorization.getRefreshToken();
if (!refreshToken.isActive()) {
		if (this.logger.isDebugEnabled()) {
		    this.logger.debug(LogMessage.format("Invalid request: refresh_token is not active for registered client '%s'", registeredClient.getId()));
    }
        throw new OAuth2AuthenticationException("invalid_grant");
    } else {
        Set<String> scopes = refreshTokenAuthentication.getScopes();
        Set<String> authorizedScopes = authorization.getAuthorizedScopes();
        if (!authorizedScopes.containsAll(scopes)) {
				    throw new OAuth2AuthenticationException("invalid_scope");
        } else {
            if (this.logger.isTraceEnabled()) {
			          this.logger.trace("Validated token request parameters");
            }
            if (scopes.isEmpty()) {
					      scopes = authorizedScopes;
            }
            DefaultOAuth2TokenContext.Builder tokenContextBuilder = (DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)((DefaultOAuth2TokenContext.Builder)DefaultOAuth2TokenContext.builder().registeredClient(registeredClient)).principal((Authentication)authorization.getAttribute(Principal.class.getName()))).authorizationServerContext(AuthorizationServerContextHolder.getContext())).authorization(authorization)).authorizedScopes(scopes)).authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)).authorizationGrant(refreshTokenAuthentication);
            OAuth2Authorization.Builder authorizationBuilder = OAuth2Authorization.from(authorization);
            OAuth2TokenContext tokenContext = ((DefaultOAuth2TokenContext.Builder)tokenContextBuilder.tokenType(OAuth2TokenType.ACCESS_TOKEN)).build();
            OAuth2Token generatedAccessToken = this.tokenGenerator.generate(tokenContext);
```

- - -

### 권한 검증
**문제점**
* `hasRole('MASTER')` 적용 x
  * `JwtAuthenticationProvider`의 token 값에 role이 밑 사진 처럼 들어가서  
  ![img.png](../../image/roleset.png)
  * SecurityExpressionRoot 의 hasAnyAuthorityName가 false 로 반환 되어 Access Denied 에러 발생 roleSet에 MASTER가 들어가 있어야 에러가 안 남

* 토큰 생성을 시큐리티에게 위임하면 roleSet에 ROLE_MASTER 가 없어서 에러 발생  
  ![img.png](../../image/authority.png)

```java
private boolean hasAnyAuthorityName(String prefix, String... roles) {
        Set<String> roleSet = this.getAuthoritySet();
        String[] var4 = roles;
        int var5 = roles.length;

        for(int var6 = 0; var6 < var5; ++var6) {
            String role = var4[var6];
            String defaultedRole = getRoleWithDefaultPrefix(prefix, role);
            if (roleSet.contains(defaultedRole)) {
                return true;
            }
        }
        return false;
}
```

**해결**
* 토큰 생성을 role도 담기도록 커스텀
  * 장점
    * db조회하지 않고 role 검증 가능 (db 커넥션 최소화)
  * 단점
    * 시큐리티 커스텀으로 코드 복잡성 증가
* 인터셉터를 활용해 db조회해서 role 검증
  * 장점
    * 토큰 생성을 커스텀하지 않아도 돼 구현 난이도 쉬움
  * 단점
    * 권한 검증이 필요할 때마다 db 조회해야함
   
  ### 인터셉터를 활용한 role 검증 구현
  ```java
  private boolean handleHandlerMethod(HandlerMethod handler) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String userId = authentication.getName();
    	// 추가 필요 로직 작성
        return true;
    }
  ```

  **동작 원리**
  1. 토큰을 검증한다.
  2. 이상이 없으면 `SecurityContextHolder`에 토큰에 담긴 값을 넣는다.
 
  **토큰 검증**  
  `X509CertificateThumbprintValidator` 클래스, `JwtTimestampValidator` 클래스에서 토큰 검증
  ``` java
  // X509CertificateThumbprintValidator 클래스
  public OAuth2TokenValidatorResult validate(Jwt jwt) {
  	// Thumbprint(인증서) 검증
  }
  // JwtTimestampValidator 클래스
  public OAuth2TokenValidatorResult validate(Jwt jwt) {
  	//토큰 만료 검증 로직
  }
  ```
