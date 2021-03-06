---
title: 리소스 및 데이터에 대한 위협 관리
titleSuffix: Azure AD B2C
description: Azure Active Directory B2C에서 서비스 거부 공격 및 암호 공격에 대한 검색 및 완화 기법을 알아봅니다.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/26/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: a438363b054361420222804dffac7973470e82e7
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78183604"
---
# <a name="manage-threats-to-resources-and-data-in-azure-active-directory-b2c"></a>Azure Active Directory B2C에서 리소스 및 데이터에 대한 위협 관리

Azure Active Directory B2C(Azure AD B2C)에는 리소스 및 데이터에 대한 위협으로부터 보호하는 데 도움이 되는 기본 제공 기능이 있습니다. 이러한 위협에는 서비스 거부 공격 및 암호 공격이 포함됩니다. 서비스 거부 공격은 의도한 사용자가 리소스를 사용할 수 없게 만듭니다. 암호 공격으로 인해 리소스에 대한 무단 액세스가 발생할 수 있습니다.

## <a name="denial-of-service-attacks"></a>서비스 거부 공격

Azure AD B2C는 SYN 쿠키를 사용하여 SYN 서비스 장애 공격을 방어합니다. Azure AD B2C는 또한 속도 및 연결 제한을 사용하여 서비스 거부 공격으로부터 보호합니다.

## <a name="password-attacks"></a>암호 공격

사용자가 설정한 암호는 적절한 복잡성을 유지해야 합니다. Azure AD B2C에는 암호 공격에 대한 완화 기술이 포함됩니다. 완화에는 무차별 암호 공격 및 사전 암호 공격 감지가 포함됩니다. 다양한 신호를 사용하여 Azure AD B2C는 요청의 무결성을 분석합니다. Azure AD B2C는 의도한 사용자와 해커 및 봇네트 간을 지능적으로 구분하도록 설계되었습니다.

Azure AD B2C는 좀 더 복잡한 계정 잠금 전략을 사용합니다. 계정은 요청의 IP 주소와 입력된 암호를 기반으로 잠깁니다. 또한 실패한 시도가 공격일 가능성에 따라 잠금 기간이 늘어납니다. 암호를 10번 시도하지 않은 경우(기본 시도 임계값) 1분 잠금이 발생합니다. 다음에 계정이 잠금 해제된 후 로그인에 실패할 때(즉, 잠금 기간이 만료되면 서비스에서 계정이 자동으로 잠금 해제된 후) 실패한 각 로그인에 대해 1분 간 다른 잠금이 발생하고 계속됩니다. 반복적으로 같은 암호를 입력해도 여러 번의 로그인 실패로 계산되지 않습니다.

처음 10회에 대한 잠금 기간은 1분입니다. 다음 10회 잠금 기간은 약간 더 길고 10회 잠금 기간마다 기간이 늘어납니다. 계정이 잠겨 있지 않은 상태에서 로그인에 성공하면 잠금 카운터가 0으로 다시 설정됩니다. 잠금 기간은 5시간까지 지속될 수 있습니다.

## <a name="manage-password-protection-settings"></a>암호 보호 설정 관리

잠금 임계값을 포함하여 암호 보호 설정을 관리하려면 다음을 수행하십시오.

1. [Azure 포털에](https://portal.azure.com) 로그인
1. 최상위 메뉴의 **디렉터리 + 구독** 필터를 사용하여 Azure AD B2C 테넌트를 포함하는 디렉터리를 선택합니다.
1. 왼쪽 메뉴에서 **Azure AD B2C**를 선택합니다. 또는 **모든 서비스**를 선택하고 **Azure AD B2C**를 검색하여 선택합니다.
1. **보안에서** **인증 방법(미리 보기)을**선택한 다음 **암호 보호를**선택합니다.
1. 원하는 암호 보호 설정을 입력한 다음 **저장을**선택합니다.

    ![Azure AD 설정의 Azure 포털 암호 보호 페이지](./media/threat-management/portal-02-password-protection.png)
    <br />* **암호 보호** 설정에서 잠금 임계값을 5로 설정합니다.*

## <a name="view-locked-out-accounts"></a>잠긴 계정 보기

잠긴 계정에 대한 정보를 얻으려면 Active Directory [로그인 활동 보고서를](../active-directory/reports-monitoring/reference-sign-ins-error-codes.md)확인할 수 있습니다. **상태**아래에서 **실패를**선택합니다. **로그인 오류 코드가** 잠긴 계정을 `50053` 나타내는 로그인 시도에 실패했습니다.

![잠긴 계정을 보여주는 Azure AD 로그인 보고서 섹션](./media/threat-management/portal-01-locked-account.png)

Azure Active Directory에서 로그인 활동 보고서를 보는 방법에 대해 자세히 알아보려면 [로그인 활동 보고서 오류 코드를](../active-directory/reports-monitoring/reference-sign-ins-error-codes.md)참조하십시오.
