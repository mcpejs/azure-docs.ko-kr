---
title: Azure 클래식 구독 관리자 | Microsoft Docs
description: Azure 공동 관리자 및 서비스 관리자 역할을 추가하거나 변경하는 방법과 계정 관리자를 확인하는 방법을 설명합니다.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/22/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 2493e893f9afda0642bd838c94538dd0b984bce5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79243485"
---
# <a name="azure-classic-subscription-administrators"></a>Azure 클래식 구독 관리자

RBAC(역할 기반 액세스 제어)를 사용하여 Azure 리소스에 대한 액세스를 관리하는 것이 좋습니다. 그러나 여전히 클래식 배포 모델을 사용 하는 경우 클래식 구독 관리자 역할( 서비스 관리자 및 공동 관리자)을 사용 해야 합니다. 자세한 내용은 [Azure 리소스 관리자 및 기존 배포를 참조하십시오.](../azure-resource-manager/management/deployment-models.md)

이 문서에서는 공동 관리자 및 서비스 관리자 역할을 추가하거나 변경하는 방법과 계정 관리자를 확인하는 방법을 설명합니다.

## <a name="add-a-co-administrator"></a>공동 관리자 추가

> [!TIP]
> 사용자가 [Azure 서비스 관리 PowerShell 모듈](https://docs.microsoft.com/powershell/module/servicemanagement/azure)을 사용하여 Azure 클래식 배포를 관리해야 하는 경우에만 공동 관리자를 추가하면 됩니다. 사용자가 Azure Portal을 통해서만 클래식 리소스를 관리하는 경우 사용자에 대한 클래식 관리자를 추가할 필요가 없습니다.

1. 서비스 관리자 또는 공동 관리자로 [Azure 포털에](https://portal.azure.com) 로그인합니다.

1. [구독](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)을 열고 구독을 선택합니다.

    공동 관리자는 구독 범위에서만 할당할 수 있습니다.

1. **액세스 제어(IAM)를 클릭합니다.**

1. **클래식 관리자** 탭을 클릭합니다.

    ![클래식 관리자를 여는 스크린샷](./media/classic-administrators/classic-administrators.png)

1. 공동 관리자 **추가를** > 클릭하여 공동**관리자** 추가 창을 엽니다.

    공동 관리자 추가 옵션을 사용할 수 없는 경우 권한이 없는 것입니다.

1. 추가할 사용자를 선택하고 **추가**를 클릭합니다.

    ![공동 관리자를 추가하는 스크린샷](./media/classic-administrators/add-coadmin.png)

## <a name="add-a-guest-user-as-a-co-administrator"></a>게스트 사용자를 공동 관리자로 추가

게스트 사용자를 공동 관리자로 추가하려면 이전 [공동 관리자 추가](#add-a-co-administrator) 섹션과 동일한 단계를 따릅니다. 게스트 사용자는 다음 기준을 충족해야 합니다.

- 게스트 사용자는 디렉터리에 있어야 합니다. 즉, 사용자가 디렉터리로 초대되어 초대를 수락했습니다.

디렉터리에 게스트 사용자를 추가하는 방법에 대한 자세한 내용은 [Azure 포털의 Azure Active Directory B2B 공동 작업 사용자 추가를](../active-directory/b2b/add-users-administrator.md)참조하십시오.

### <a name="differences-for-guest-users"></a>게스트 사용자의 차이점

공동 관리자 역할이 할당된 게스트 사용자는 공동 관리자 역할을 가진 구성원 사용자와 비교하여 약간의 차이점을 볼 수 있습니다. 다음과 같은 시나리오를 고려해 보세요.

- Azure AD 계정(직장 또는 학교 계정)이 있는 사용자 A는 Azure 구독의 서비스 관리자입니다.
- 사용자 B에게는 Microsoft 계정이 있습니다.
- 사용자 A가 사용자 B에게 공동 관리자 역할을 할당합니다.
- 사용자 B는 거의 모든 작업을 수행할 수 있지만 Azure AD 디렉터리에서 애플리케이션을 등록하거나 사용자를 조회할 수 없습니다.

사용자 B는 모든 것을 관리할 수 있다고 예상할 수 있습니다. 이러한 차이가 나타나는 이유는 Microsoft 계정이 멤버 사용자가 아닌 게스트 사용자로 구독에 추가되기 때문입니다. 게스트 사용자에 멤버 사용자와 비교할 때 Azure AD에서 다른 기본 권한을 갖습니다. 예를 들어, 멤버 사용자는 Azure AD에서 다른 사용자가 읽을 수 있지만 게스트 사용자는 그럴 수 없습니다. 멤버 사용자는 Azure AD에서 새 서비스 주체를 등록할 수 있지만 게스트 사용자는 그럴 수 없습니다.

게스트 사용자가 이러한 작업을 수행할 수 있도록 하려면 게스트 사용자가 필요한 특정 Azure AD 관리자 역할을 할당해야 합니다. 예를 들어, 이전 시나리오에서는 다른 사용자를 읽기 위한 [디렉터리 읽기 권한자](../active-directory/users-groups-roles/directory-assign-admin-roles.md#directory-readers) 역할과 서비스 주체를 만들 수 있는 [애플리케이션 개발자](../active-directory/users-groups-roles/directory-assign-admin-roles.md#application-developer) 역할을 할당할 수 있습니다. 멤버 및 게스트 사용자와 해당 권한에 대한 자세한 내용은 [Azure Active Directory의 기본 사용자 권한이란?](../active-directory/fundamentals/users-default-permissions.md)을 참조하세요. 게스트 사용자에 대한 액세스 권한 부여에 대한 자세한 내용은 [RBAC를 사용하는 외부 게스트 사용자의 Azure 리소스에 대한 액세스 관리를](role-assignments-external-users.md)참조하십시오.

[Azure 리소스의 기본 제공 역할](../role-based-access-control/built-in-roles.md)은 [Azure AD 관리자 역할](../active-directory/users-groups-roles/directory-assign-admin-roles.md)과 다릅니다. 기본 제공 역할은 Azure AD에 대한 액세스 권한을 부여하지 않습니다. 자세한 내용은 [다른 역할 이해](../role-based-access-control/rbac-and-directory-admin-roles.md)를 참조하세요.

멤버 사용자와 게스트 사용자를 비교하는 정보는 [Azure Active Directory의 기본 사용자 사용 권한은 무엇입니까?](../active-directory/fundamentals/users-default-permissions.md)

## <a name="remove-a-co-administrator"></a>공동 관리자 제거

1. 서비스 관리자 또는 공동 관리자로 [Azure 포털에](https://portal.azure.com) 로그인합니다.

1. [구독](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)을 열고 구독을 선택합니다.

1. **액세스 제어(IAM)를 클릭합니다.**

1. **클래식 관리자** 탭을 클릭합니다.

1. 제거할 공동 관리자 옆에 확인 표시를 추가합니다.

1. **제거**를 클릭합니다.

1. 표시되는 메시지 상자에서 **예**를 클릭합니다.

    ![공동 관리자를 제거하는 스크린샷](./media/classic-administrators/remove-coadmin.png)

## <a name="change-the-service-administrator"></a>서비스 관리자 변경

계정 관리자만 구독에 대한 서비스 관리자를 변경할 수 있습니다. 기본적으로, Azure 구독에 가입할 때 서비스 관리자는 계정 관리자와 같습니다. 계정 관리자 역할의 사용자는 Azure Portal에 액세스할 수 없습니다. 서비스 관리자 역할의 사용자는 Azure Portal에 대한 모든 권한이 있습니다. 계정 관리자 및 서비스 관리자가 동일한 사용자이고 서비스 관리자를 다른 사용자로 변경하는 경우 계정 관리자는 Azure Portal에 액세스할 수 없게 됩니다. 그러나 계정 관리자는 계정 센터를 사용하여 언제든지 서비스 관리자를 다시 자신으로 변경할 수 있습니다.

다음 단계에 따라 **계정 센터의**서비스 관리자를 변경합니다.

### <a name="account-center"></a>계정 센터

1. 서비스 관리자 를 [변경하기 위한 제한 사항을](#limitations-for-changing-the-service-administrator)확인하여 시나리오가 지원되는지 확인합니다.

1. 계정 관리자로 [계정 센터에](https://account.windowsazure.com/subscriptions) 로그인합니다.

1. 구독을 클릭합니다.

1. 오른쪽에서 **구독 세부 정보 편집을 클릭합니다.**

    ![계정 센터에서 구독 편집 단추를 보여 주는 스크린샷](./media/classic-administrators/editsub.png)

1. 서비스 **관리자** 상자에 새 서비스 관리자의 전자 메일 주소를 입력합니다.

    ![서비스 관리자 전자 메일을 변경하는 상자를 보여 주는 스크린샷](./media/classic-administrators/change-service-admin.png)

1. 확인 표시를 클릭하여 변경 내용을 저장합니다.

### <a name="limitations-for-changing-the-service-administrator"></a>서비스 관리자 변경에 대한 제한 사항

Azure 구독당 서비스 관리자는 하나만 있을 수 있습니다. 서비스 관리자 변경은 계정 관리자가 Microsoft 계정인지 또는 Azure AD 계정(직장 또는 학교 계정)인지여부에 따라 다르게 작동합니다.

| 계정 관리자 계정 | 서비스 관리자를 다른 Microsoft 계정으로 변경할 수 있습니까? | 서비스 관리자를 동일한 디렉터리에서 Azure AD 계정으로 변경할 수 있습니까? | 서비스 관리자를 다른 디렉터리에서 Azure AD 계정으로 변경할 수 있습니까? |
| --- | --- | --- | --- |
| Microsoft 계정 | yes | 예 | 예 |
| Azure AD 계정 | yes | yes | 예 |

계정 관리자가 Azure AD 계정인 경우 서비스 관리자를 동일한 디렉터리에서 Azure AD 계정으로 변경할 수 있지만 다른 디렉터리에서는 변경할 수 없습니다. 예를 들어 abby@contoso.com 서비스 관리자를 bob@contoso.com로 변경할 수 있지만 john@notcontoso.com contoso.com john@notcontoso.com 디렉터리에 있는 경우가 아니면 서비스 관리자를 변경할 수 없습니다.

Microsoft 계정 및 Azure AD 계정에 대한 자세한 내용은 [Azure Active Directory란 무엇입니까?](../active-directory/fundamentals/active-directory-whatis.md)

## <a name="view-the-account-administrator"></a>계정 관리자 보기

계정 관리자는 초기에 Azure 구독에 등록한 사용자이며 구독의 청구 소유자로서 책임이 있습니다. 구독의 계정 관리자를 변경하려면 [Azure 구독의 소유권을 다른 계정으로 이전합니다.](../cost-management-billing/manage/billing-subscription-transfer.md)

계정 관리자를 보려면 다음 단계를 수행합니다.

1. [Azure 포털에](https://portal.azure.com)로그인합니다.

1. [구독](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)을 열고 구독을 선택합니다.

1. **속성**을 클릭합니다.

    구독의 계정 관리자가 계정 **관리자** 상자에 표시됩니다.

    ![계정 관리자를 보여 주는 스크린샷](./media/classic-administrators/account-admin.png)

## <a name="next-steps"></a>다음 단계

* [Azure의 다양한 역할 이해](../role-based-access-control/rbac-and-directory-admin-roles.md)
* [RBAC 및 Azure Portal을 사용하여 Azure 리소스에 대한 액세스 관리](../role-based-access-control/role-assignments-portal.md)
* [Azure 구독 관리자 추가 또는 변경](../cost-management-billing/manage/add-change-subscription-administrator.md)
