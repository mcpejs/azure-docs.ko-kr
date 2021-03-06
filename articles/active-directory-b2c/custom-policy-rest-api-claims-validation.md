---
title: REST API 클레임 교환을 유효성 검사로
titleSuffix: Azure AD B2C
description: RESTful 서비스와 상호 작용하는 Azure AD B2C 사용자 여정을 만들기 위한 연습입니다.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/26/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: a4902e96cd41a02953b6686b5d52d7912b27809f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80330831"
---
# <a name="walkthrough-integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journey-to-validate-user-input"></a>연습: Azure AD B2C 사용자 여정에서 REST API 클레임 교환통합하여 사용자 입력의 유효성을 검사합니다.

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C(Azure AD B2C)를 뒷받침하는 IEF(ID 환경 프레임워크)를 사용하면 ID 개발자가 사용자 여정에서 RESTful API와 상호 작용을 통합할 수 있습니다.  이 연습의 끝에서 [RESTful 서비스와](custom-policy-rest-api-intro.md) 상호 작용하는 Azure AD B2C 사용자 여정을 만들어 사용자 입력의 유효성을 검사할 수 있습니다.

이 시나리오에서는 사용자가 Azure AD B2C 가입 페이지에 충성도 번호를 입력하는 기능을 추가합니다. 이 데이터를 REST API로 전송하여 이메일과 충성도 번호의 조합이 프로모션 코드에 매핑되는지 여부를 확인합니다. REST API에서 이 사용자에 대한 프로모션 코드를 찾으면 Azure AD B2C로 반환됩니다. 마지막으로, 프로모션 코드는 응용 프로그램이 사용할 토큰 클레임에 삽입됩니다.

또한 상호 작용은 오케스트레이션 단계로도 설계할 수 있습니다. 이는 REST API가 화면에서 데이터를 검증하지 않고 항상 클레임을 반환하는 경우에 적합합니다. 자세한 내용은 [연습: Azure AD B2C 사용자 경험에서 REST API 클레임 교환을 오케스트레이션 단계로 통합](custom-policy-rest-api-claims-exchange.md)을 참조하세요.

## <a name="prerequisites"></a>사전 요구 사항

- [사용자 지정 정책 시작](custom-policy-get-started.md)의 단계를 완료합니다. 로컬 계정을 사용하여 등록 및 로그인하기 위한 사용자 지정 정책이 작동해야 합니다.
- [Azure AD B2C 사용자 지정 정책에서 REST API 클레임 교환을 통합하는](custom-policy-rest-api-intro.md)방법을 알아봅니다.

## <a name="prepare-a-rest-api-endpoint"></a>REST API 끝점 준비

이 연습에서는 이메일 주소가 로열티 ID를 사용하여 백 엔드 시스템에 등록되어 있는지 여부를 확인하는 REST API가 있어야 합니다. 등록된 경우 REST API는 고객이 응용 프로그램 내에서 상품을 구매하는 데 사용할 수 있는 등록 프로모션 코드를 반환해야 합니다. 그렇지 않으면 REST API는 HTTP 409 오류 메시지를 반환해야 합니다: "로열티 ID '{충성도 ID}' '{email}' 이메일 주소와 연결되지 않습니다.".

다음 JSON 코드는 Azure AD B2C가 REST API 끝점으로 보내는 데이터를 보여 줍니다. 

```json
{
    "email": "User email address",
    "language": "Current UI language",
    "loyaltyId": "User loyalty ID"
}
```

REST API가 데이터의 유효성을 검사하면 다음 JSON 데이터와 함께 HTTP 200(확인)을 반환해야 합니다.

```json
{
    "promoCode": "24534"
}
```

유효성 검사가 실패하면 REST API는 JSON 요소와 함께 `userMessage` HTTP 409(충돌)를 반환해야 합니다. IEF는 REST `userMessage` API가 반환하는 클레임을 기대합니다. 유효성 검사가 실패하면 이 클레임은 사용자에게 문자열로 표시됩니다.

```json
{
    "version": "1.0.1",
    "status": 409,
    "userMessage": "LoyaltyId ID '1234' is not associated with 'david@contoso.com' email address."
}
```

REST API 끝점의 설정은 이 문서의 범위를 벗어납니다. [Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-reference) 샘플을 만들었습니다. [GitHub](https://github.com/azure-ad-b2c/rest-api/tree/master/source-code/azure-function)에서 전체 Azure 함수 코드에 액세스할 수 있습니다.

## <a name="define-claims"></a>클레임 정의

클레임은 Azure AD B2C 정책 실행 중에 데이터의 임시 저장소를 제공합니다. [클레임 스키마](claimsschema.md) 섹션에서 클레임을 선언할 수 있습니다. 

1. 정책의 확장 파일을 엽니다. 예를 들어, <em> `SocialAndLocalAccounts/` </em>.
1. [BuildingBlocks](buildingblocks.md) 요소를 검색합니다. 요소가 존재하지 않는 경우 추가합니다.
1. [클레임스키마](claimsschema.md) 요소를 찾습니다. 요소가 존재하지 않는 경우 추가합니다.
1. **클레임스키마** 요소에 다음 클레임을 추가합니다.  

```xml
<ClaimType Id="loyaltyId">
  <DisplayName>Your loyalty ID</DisplayName>
  <DataType>string</DataType>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
<ClaimType Id="promoCode">
  <DisplayName>Your promo code</DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
</ClaimType>
  <ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="configure-the-restful-api-technical-profile"></a>RESTful API 기술 프로필 구성 

[나머지 기술 프로필은](restful-technical-profile.md) 자신의 RESTful 서비스에 대한 인터페이싱을 지원합니다. Azure AD B2C는 `InputClaims` 컬렉션의 RESTful 서비스에 데이터를 보내고 `OutputClaims` 컬렉션에서 데이터를 다시 수신합니다. 클레임 **공급자** 요소를 찾아 다음과 같이 새 클레임 공급자를 추가합니다.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Check loyaltyId Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/ValidateProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="loyaltyId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

이 예제에서는 `userLanguage` JSON 페이로드 내에서REST `lang` 서비스로 전송됩니다. 클레임 값에는 `userLanguage` 현재 사용자 언어 ID가 포함되어 있습니다. 자세한 내용은 [클레임 해결 을](claim-resolver-overview.md)참조하십시오.

위의 `AuthenticationType` 주석과 `AllowInsecureAuthInProduction` 프로덕션 환경으로 이동할 때 수행해야 하는 변경 내용을 지정합니다. 프로덕션을 위해 RESTful API를 보호하는 방법에 대해 알아보려면 [보안 RESTful API를](secure-rest-api.md)참조하십시오.

## <a name="validate-the-user-input"></a>사용자 입력 유효성 검사

가입 하는 동안 사용자의 충성도 번호를 얻으려면 사용자가 화면에이 데이터를 입력 하도록 허용 해야 합니다. 기존 등록 기술 프로필 섹션의 `OutputClaims` 요소에 추가하여 등록 페이지에 **loyaltyId** 출력 클레임을 추가합니다. 출력 클레임의 전체 목록을 지정하여 클레임이 화면에 표시되는 순서를 제어합니다.  

을 호출하는 등록 기술 프로필에 유효성 검사 기술 `REST-ValidateProfile`프로필 참조를 추가합니다. 새 유효성 검사 기술 프로필은 기본 `<ValidationTechnicalProfiles>` 정책에 정의된 컬렉션의 맨 위에 추가됩니다. 이 동작은 유효성 검사가 성공한 후에만 Azure AD B2C가 디렉터리에서 계정을 만들기 위해 이동한다는 것을 의미합니다.   

1. **ClaimsProviders** 요소를 찾습니다. 다음과 같이 새 클레임 공급자를 추가합니다.

    ```xml
    <ClaimsProvider>
      <DisplayName>Local Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="newPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true"/>
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surName"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile" />
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    <ClaimsProvider>
      <DisplayName>Self Asserted</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SelfAsserted-Social">
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="email" />
          </InputClaims>
            <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="displayName"/>
            <OutputClaim ClaimTypeReferenceId="givenName"/>
            <OutputClaim ClaimTypeReferenceId="surname"/>
            <!-- Required to present the text box to collect the data from the user -->
            <OutputClaim ClaimTypeReferenceId="loyaltyId"/>
            <!-- Required to pass the promoCode returned from "REST-ValidateProfile" 
            to subsequent orchestration steps and token issuance-->
            <OutputClaim ClaimTypeReferenceId="promoCode" />
          </OutputClaims>
          <ValidationTechnicalProfiles>
            <ValidationTechnicalProfile ReferenceId="REST-ValidateProfile"/>
          </ValidationTechnicalProfiles>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

## <a name="include-a-claim-in-the-token"></a>토큰에 클레임 포함 

프로모션 코드 클레임을 다시 사용 당사자 응용 프로그램으로 반환하려면 <em> `SocialAndLocalAccounts/` </em> 파일에 출력 클레임을 추가합니다. 출력 클레임은 성공적인 사용자 여정 후에 클레임을 토큰에 추가할 수 있도록 허용하고 응용 프로그램으로 전송됩니다. 의존 자 섹션 내의 기술 프로파일 요소를 `promoCode` 수정하여 출력 클레임으로 추가합니다.
 
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <OutputClaim ClaimTypeReferenceId="promoCode" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="test-the-custom-policy"></a>사용자 지정 정책 테스트

1. [Azure 포털에](https://portal.azure.com)로그인합니다.
1. 최상위 메뉴에서 **디렉터리 + 구독** 필터를 선택하고 Azure AD 테넌트가 포함된 디렉터리를 선택하여 Azure AD 테넌트를 포함하는 디렉터리를 사용하고 있는지 확인합니다.
1. Azure Portal의 왼쪽 상단 모서리에서 **모든 서비스**를 선택한 다음, **앱 등록**을 검색하여 선택합니다.
1. **ID 경험 프레임워크**를 선택합니다.
1. **사용자 지정 정책 업로드를**선택한 다음 변경한 정책 파일인 *TrustFrameworkExtensions.xml*및 *SignUpOrSignin.xml*을 업로드합니다. 
1. 업로드한 등록 또는 로그인 정책을 선택하고 **지금 실행** 단추를 클릭합니다.
1. 전자 메일 주소를 사용하여 등록할 수 있습니다.
1. **지금 등록** 링크를 클릭합니다.
1. 로열티 ID에서 1234를 입력하고 **계속을** **클릭합니다.** 이 시점에서 유효성 검사 오류 메시지가 나타납니다.
1. 다른 값으로 변경하고 **계속을**클릭합니다.
1. 애플리케이션으로 다시 전송되는 토큰에는 `promoCode` 클레임이 포함됩니다.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584295703,
  "nbf": 1584292103,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584292103,
  "auth_time": 1584292103,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "promoCode": "84362"
  ...
}
```

## <a name="next-steps"></a>다음 단계

API를 보호하는 방법을 알아보려면 다음 문서를 참조하십시오.

- [연습: Azure AD B2C 사용자 경험에서 REST API 클레임 교환을 오케스트레이션 단계로 통합](custom-policy-rest-api-claims-exchange.md)
- [리시풀 API 보안](secure-rest-api.md)
- [참조: RESTful 기술 프로필](restful-technical-profile.md)
