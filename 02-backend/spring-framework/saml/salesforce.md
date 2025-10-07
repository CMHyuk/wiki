## Salesforce Sp 적용
- **issuer**에는 **IdP의 URL**을 입력하여 IdP를 식별
- **Entity ID**에는 **SP의 URL**을 입력하여 SAML assertion이 어느 SP를 대상으로 하는지 명시

#### Salesforce SAML Response example

```xml
<samlp:Response Destination="https://MyDomainName.my.salesforce.com?so=00Dx00000000001"
  ID="_0f551f9288c8b76f21c3d4d15c9cd1df1290476801091"
  InResponseTo="_2INwHuINDJTvjo8ohcM.Fpw_uLukYi0WArVx2IJD569kZYL
    osBwuiaSbzzxOPQjDtfw52tJB10VfgPW2p5g7Nlv5k1QDzR0EJYGgn0d0z8
    CIiUOY31YBdk7gwEkTygiK_lb46IO1fzBFoaRTzwvf1JN4qnkGttw3J6L4b
    opRI8hSQmCumM_Cvn3DHZVN.KtrzzOAflcMFSCY.bj1wvruSGQCooTRSSQ"
  IssueInstant="2010-11-23T01:46:41.091Z" Version="2.0">
<saml:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity"
>https://myidp.my.salesforce.com</saml:Issuer>
−
<ds:Signature>
−
<ds:SignedInfo>
<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
<ds:SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1"/>
−
<ds:Reference URI="#_0f551f9288c8b76f21c3d4d15c9cd1df1290476801091">
−
<ds:Transforms>
<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
−
<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#">
<ec:InclusiveNamespaces PrefixList="ds saml samlp xs"/>
</ds:Transform>
</ds:Transforms>
<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
<ds:DigestValue>4NVTbQ2WavD+ZBiyQ7ufc8EhtZw=</ds:DigestValue>
</ds:Reference>
</ds:SignedInfo>
-
<ds:SignatureValue>

ExampleSamlSignature</ds:SignatureValue>
−
<ds:KeyInfo>
−
<ds:X509Data>
−
<ds:X509Certificate>ExampleX509 Certificate</ds:X509Certificate>
</ds:X509Data>
</ds:KeyInfo>
</ds:Signature>
−
<samlp:Status>
<samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
</samlp:Status>
−
<saml:Assertion ID="_e700bf9b25a5aebdb9495fe40332ef081290476801092" IssueInstant="2010-11-23T01:46:41.092Z" Version="2.0">
<saml:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity">https://exampleidp.com</saml:Issuer>
−
<saml:Subject>
<saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">exampleuser@salesforce.com</saml:NameID>
−
<saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
<saml:SubjectConfirmationData NotOnOrAfter="2010-11-23T01:51:41.093Z" Recipient="https://MyDomainName.my.salesforce.com?so=00Dx00000000001"/>
</saml:SubjectConfirmation>
</saml:Subject>
−
<saml:Conditions NotBefore="2010-11-23T01:46:41.093Z" NotOnOrAfter="2010-11-23T01:51:41.093Z">
−
<saml:AudienceRestriction>
<saml:Audience>https://exampleserviceprovider.com</saml:Audience>
</saml:AudienceRestriction>
</saml:Conditions>
−
<saml:AuthnStatement AuthnInstant="2010-11-23T01:46:41.092Z">
−
<saml:AuthnContext>
<saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified</saml:AuthnContextClassRef>
</saml:AuthnContext>
</saml:AuthnStatement>
−
<saml:AttributeStatement>
−
<saml:Attribute Name="userId" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
<saml:AttributeValue xsi:type="xs:anyType">005D0000001Ayzh</saml:AttributeValue>
</saml:Attribute>
−
<saml:Attribute Name="username" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
<saml:AttributeValue xsi:type="xs:anyType">admin@identity.org</saml:AttributeValue>
</saml:Attribute>
−
<saml:Attribute Name="email" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
<saml:AttributeValue xsi:type="xs:anyType">exampleuser@salesforce.com</saml:AttributeValue>
</saml:Attribute>
−
<saml:Attribute Name="is_portal_user" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
<saml:AttributeValue xsi:type="xs:anyType">false</saml:AttributeValue>
</saml:Attribute>
</saml:AttributeStatement>
</saml:Assertion>
</samlp:Response>
```

### 참고
공식 문서

https://help.salesforce.com/s/articleView?id=sf.sso_saml.htm&type=5

SAML SSO 설정 링크 (로그인 필요)

https://saas-customer-8145.lightning.force.com/lightning/setup/SingleSignOn/page?address=%2F0LE%2Fe%3FretURL%3D%252F_ui%252Fidentity%252Fsaml%252FSingleSignOnSettingsUi%252Fd