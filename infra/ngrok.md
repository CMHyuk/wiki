## Ngrok
* 로컬 개발 환경에서 인터넷을 통해 웹 애플리케이션에 안전하게 접근할 수 있도록 해주는 도구  
* 보안 연결을 통해 인터넷에서 서버를 실행할 수 있으며, 웹 애플리케이션을 외부에 노출시키지 않고도 테스트할 수 있다.

### 사용 방법

#### 다운로드
https://ngrok.com/download OS에 맞게 다운로드 한다.

#### 토큰 등록
`ngrok config add-authtoken {Token 값}`

#### 도메인 연결
* 로컬 서버 포트가 8080일 때  
  * `ngrok http 8080`

* 여러 개의 도메인을 연결
  * version 3의 경우 유로 결제를 해야한다.
```yaml
version: "2"
authtoken: 23oJkcJMk8EV9xgBRs2AoisGLfP_6speGduSgaEG9QShjaZuq
region: jp
tunnels:
  saml-idp:
    addr: http://localhost:9001
    proto: http
    hostname: saml-idp-service.jp.ngrok.io
  idgp-service:
    addr: http://localhost:9000
    proto: http
    hostname: oauth-service.jp.ngrok.io
```

* 설정 파일 적용
`ngrok start --config "{경로}" --all`
  * 경로 예시) `C:\Windows\ngrok.yml`
