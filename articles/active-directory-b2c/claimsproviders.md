---
title: ClaimsProviders  - Azure Active Directory B2C | Microsoft Docs
description: Azure Active Directory B2C에서 사용자 지정 정책의 ClaimsProvider 요소를 지정하는 방법을 설명합니다.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 01/29/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: dfb34085181e0b759d1d77485ff21b5bc59e0de3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78189772"
---
# <a name="claimsproviders"></a>ClaimsProviders

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

클레임 공급자에는 [기술 프로필](technicalprofiles.md) 집합이 포함되어 있습니다. 모든 클레임 공급자에는 엔드포인트를 결정하는 하나 이상의 기술 프로필 및 클레임 공급자와 통신하는 데 필요한 프로토콜이 있어야 합니다. 하나의 클레임 공급자가 여러 개의 기술 프로필을 포함할 수 있습니다. 예를 들어, 클레임 공급자가 여러 프로토콜, 다른 기능을 가진 다양한 엔드포인트를 지원하거나 여러 보증 수준에서 다양한 클레임을 해제하므로 여러 기술 프로필을 정의할 수 있습니다. 다른 사용자 경험이 아닌 하나의 사용자 경험에서 중요한 클레임을 해제하기 위해 허용할 수도 있습니다.

```XML
<ClaimsProviders>
  <ClaimsProvider>
    <Domain>Domain name</Domain>
    <DisplayName>Display name</DisplayName>
    <TechnicalProfiles>
      </TechnicalProfile>
        ...
      </TechnicalProfile>
        ...
    </TechnicalProfiles>
  </ClaimsProvider>
  ...
</ClaimsProviders>
```

**ClaimsProviders** 요소에는 다음과 같은 요소가 포함됩니다.

| 요소 | 발생 수 | 설명 |
| ------- | ----------- | ----------- |
| ClaimsProvider | 1:n | 다양한 사용자 journeys에서 활용할 수 있는 인증된 청구 공급자입니다. |

## <a name="claimsprovider"></a>ClaimsProvider

**ClaimsProvider** 요소에는 다음과 같은 하위 요소가 포함됩니다.

| 요소 | 발생 수 | 설명 |
| ------- | ---------- | ----------- |
| 도메인 | 0:1 | 클레임 공급자의 도메인 이름을 포함하는 문자열입니다. 예를 들어, 클레임 공급자에 Facebook 기술 프로필이 포함된 경우 도메인 이름은 Facebook.com입니다. 이 도메인 이름은 기술 프로필에 의해 대체되지 않는 한, 청구 제공자에 정의된 모든 기술 프로필에 사용됩니다. 도메인 이름은 **domain_hint**에서 참조할 수도 있습니다. 자세한 내용은 [Azure Active Directory B2C를 사용하여 직접 로그인 설정](direct-signin.md) 섹션의 **소셜 공급자로 로그인 리디렉션** 섹션을 참조하세요. |
| DisplayName | 1:1 | 클레임 공급자의 이름을 포함하는 문자열입니다. |
| [TechnicalProfiles](technicalprofiles.md) | 0:1 | 클레임 공급자가 지원하는 기술 프로필 세트입니다. |

**클레임공급자는** 기술 프로필이 클레임 공급자와 어떻게 관련되는지 구성합니다. 다음 예에서는 Azure Active Directory 기술 프로필을 사용하여 Azure Active Directory 클레임 공급자를 보여 줍니다.

```XML
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AAD-Common">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId-NoError">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
      ...
    </TechnicalProfile>
      ...
    <TechnicalProfile Id="AAD-UserWritePasswordUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      ...
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWritePhoneNumberUsingObjectId">
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

다음 예제는 **Facebook-OAUTH** 기술 프로필을 사용하여 Facebook 클레임 공급자를 보여 줍니다.

```XML
<ClaimsProvider>
  <Domain>facebook.com</Domain>
  <DisplayName>Facebook</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Facebook-OAUTH">
      <DisplayName>Facebook</DisplayName>
      <Protocol Name="OAuth2" />
        ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```
