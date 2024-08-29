Pod가 여러 개인 경우, Spring Security에서 세션 관리를 하지 않으면, 라운드 로빈 방식의 로드 밸런싱으로 인해 각 요청이 서로 다른 Pod에 분산될 수 있어 로그인이 동작되지 않기 때문에 redis에 SecurityContext를 저장해야한다.  

아래 순서대로 SecurityContext가 redis에 저장된다.

`ExceptionTranslationFilter` 클래스

```java
protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, AuthenticationException reason) throws ServletException, IOException {
    SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
    this.securityContextHolderStrategy.setContext(context);
    this.requestCache.saveRequest(request, response);
    this.authenticationEntryPoint.commence(request, response, reason); // RedisSessionRepository save 호출
}
```

`OnCommittedResponseWrapper` 클래스

```java
private void doOnResponseCommitted() {
   if (!this.disableOnCommitted) {
       this.onResponseCommitted(); //SessionRepositoryFilter의 onResponseCommitted 호출
       this.disableOnResponseCommitted();
   }
}

// SessionRepositoryFilter 클래스
private final class SessionRepositoryResponseWrapper extends OnCommittedResponseWrapper {
    private final SessionRepositoryFilter<S>.SessionRepositoryRequestWrapper request;

    SessionRepositoryResponseWrapper(SessionRepositoryFilter<S>.SessionRepositoryRequestWrapper request, HttpServletResponse response) {
        super(response);
        if (request == null) {
            throw new IllegalArgumentException("request cannot be null");
        } else {
            this.request = request;
        }
    }

    protected void onResponseCommitted() {
        this.request.commitSession();
    }
}
```

`SessionRepositoryFilter` 클래스

```java
private void commitSession() {
   SessionRepositoryFilter<S>..SessionRepositoryRequestWrapper.HttpSessionWrapper wrappedSession = this.getCurrentSession();
   if (wrappedSession == null) {
       if (this.isInvalidateClientSession()) {
           SessionRepositoryFilter.this.httpSessionIdResolver.expireSession(this, this.response);
       }
   } else {
       S session = wrappedSession.getSession();
       String requestedSessionId = this.getRequestedSessionId();
       this.clearRequestedSessionCache();
       SessionRepositoryFilter.this.sessionRepository.save(session); // redis에 SecurityContext 저장
       String sessionId = session.getId();
       if (!this.isRequestedSessionIdValid() || !sessionId.equals(requestedSessionId)) {
           SessionRepositoryFilter.this.httpSessionIdResolver.setSessionId(this, this.response, sessionId);
       }
   }
}
```

`RedisSessionRepository` 클래스 - redis에 SecurityContext 저장

```java
public void save(RedisSession session) {
   if (!session.isNew) {
        String key = this.getSessionKey(session.hasChangedSessionId() ? session.originalSessionId : session.getId());
         Boolean sessionExists = this.sessionRedisOperations.hasKey(key);
         if (sessionExists == null || !sessionExists) {
             throw new IllegalStateException("Session was invalidated");
         }
     }
     session.save();
}

private void save() {
   this.saveChangeSessionId();
   this.saveDelta();
   if (this.isNew) {
       this.isNew = false;
   }
}

private void saveDelta() {
    if (!this.delta.isEmpty()) {
        String key = RedisSessionRepository.this.getSessionKey(this.getId());
        RedisSessionRepository.this.sessionRedisOperations.opsForHash().putAll(key, new HashMap(this.delta));
        RedisSessionRepository.this.sessionRedisOperations.expireAt(key, Instant.ofEpochMilli(this.getLastAccessedTime().toEpochMilli()).plusSeconds(this.getMaxInactiveInterval().getSeconds()));
        this.delta.clear();
     }
}
```
