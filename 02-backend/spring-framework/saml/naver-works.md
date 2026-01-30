### 개요
사용자 경험과 보안성을 향상시키기 위해, 사용자가 하나의 서비스에서 로그아웃하면 연동된 모든 서비스에서도 자동으로 로그아웃되도록 SAML Single Logout(SLO) 기능을 구현하고 있었다.

SAML Single Logout의 흐름은 아래와 같다.

![img.png](../../../../assets/images/sp-initiated-single-logout.png)

체이닝 방식에 해당하는 3 ~ 7번 단계는 일단 제외하고, 최초로 로그아웃 요청을 보낸 서비스 제공자(SP)에서의 로그아웃 동작이 정상적으로 이루어지는지 확인하기 위해 네이버웍스에 SLO 설정을 적용하고 테스트를 수행했다.

---

### 문제 상황

네이버웍스에서 전달받은 SAML LogoutRequest는 디코딩 이전에 OpenSAML 라이브러리의 내부 검증 로직을 거치게되는데, SecurityException이 발생했다.

![img.png](../../../../assets/images/saml-request-code.png)

에러가 발생한 이유는, LogoutRequest는 서명된 상태(bindingRequires)로 전달되었지만 Destination 속성 값이 존재하지 않았기 때문이다.  

해당 로직을 커스터마이징하여 에러가 발생하지 않도록 처리할 수도 있겠지만, 서명된 메시지라고 해서 무조건 신뢰할 수 있는 것은 아니며, 그것이 실제로 어떤 대상에게 전송되었는지(Destination)를 검증하지 않으면 공격자가 이를 악용할 가능성이 있다.  

이러한 보안적 이유로 해당 로직이 포함된 것으로 보이며, SAML의 공식 표준화 기구인 OASIS의 [공식 문서](https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf#page=19)에서도 아래와 같은 내용을 명시하고 있다.

![img.png](../../../../assets/images/saml-oasis.png)

서명된 메시지인 경우 `Destination` 속성은 필수로 포함되어야 한다는 내용이다. 하지만 네이버웍스의 LogoutRequest는 다음과 같다.

```xml
<saml2p:LogoutRequest xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
                      ID="jjecdhkeonpkidhibdefgphkobpfchbmmenonpnk"
                      IssueInstant="2025-06-16T01:00:09.981Z"
                      Version="2.0">
    <saml2:Issuer xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">worksmobile.com</saml2:Issuer>
    <saml2:NameID xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                  Format="urn:oasis:names:tc:SAML:1.1:nameidformat:unspecified">naverworks@email.com
    </saml2:NameID>
</saml2p:LogoutRequest>
```

해당 메시지는 서명된 채로 넘어오는데 속성 값에 `Destination`이 존재하지 않았다. 공식 문서대로라면 다음과 같은 형태가 되어야 할 것이다.

```xml
<saml2p:LogoutRequest xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
                      ID="jjecdhkeonpkidhibdefgphkobpfchbmmenonpnk"
                      Destination="https://destination.com"
                      IssueInstant="2025-06-16T01:00:09.981Z"
                      Version="2.0">
    <saml2:Issuer xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion">worksmobile.com</saml2:Issuer>
    <saml2:NameID xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
                  Format="urn:oasis:names:tc:SAML:1.1:nameidformat:unspecified">naverworks@email.com
    </saml2:NameID>
</saml2p:LogoutRequest>
```

---

### 해결

해당 내용을 네이버웍스 측에 문의한 결과, 스펙 상의 오류임을 확인하였으며, 해당 사항은 내부적으로 수정하겠다는 회신을 받았다.

![img.png](../../../../assets/images/naverworks.png)