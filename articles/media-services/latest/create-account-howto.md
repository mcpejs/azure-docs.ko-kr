---
title: Azure Media Services 계정 만들기
description: 이 자습서에서는 Azure Media Services 계정을 만드는 단계를 안내합니다.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/15/2020
ms.author: juliako
ms.openlocfilehash: 2f1694825319ed8b8682c044e7e2282ed4c43dcd
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79478800"
---
# <a name="create-a-media-services-account"></a>Media Services 계정 만들기

Azure에서 암호화, 인코딩, 분석, 관리 및 스트리밍을 시작하려면 Media Services 계정을 만들어야 합니다. Media Services 계정은 하나 이상의 스토리지 계정과 연결되어야 합니다.

> [!NOTE]
> Media Services 계정 및 연결된 모든 스토리지 계정은 동일한 Azure 구독에 포함되어야 합니다. 추가 대기 시간 및 데이터 송신 비용이 발생하지 않도록 Media Services 계정과 동일한 위치에 있는 스토리지 계정을 사용하는 것이 좋습니다.

이 문서에서는 새 Azure Media Services 계정을 만드는 단계를 설명합니다. 다음 탭에서 선택합니다.

## <a name="use-the-azure-portal"></a>Azure Portal 사용

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

Azure 포털은 Azure 미디어 서비스 계정을 빠르게 만드는 방법을 제공합니다. 계정을 사용하여 Azure에서 미디어 콘텐츠를 저장, 암호화, 인코딩, 관리 및 스트리밍할 수 있는 Media Services에 액세스할 수 있습니다.

현재 [Azure 포털을](https://portal.azure.com/) 사용하여 다음을 수행할 수 있습니다.

* 미디어 서비스 v3 [라이브 이벤트](live-events-outputs-concept.md)관리 , 
* 뷰(관리 안 됨) v3 [자산](assets-concept.md), 
* [API 에 액세스하는 것에 대한 정보를 얻을 수 있습니다.](access-api-portal.md) 

다른 모든 관리 작업(예: [변환 및 작업](transforms-jobs-concept.md) 및 콘텐츠 [보호)의](content-protection-overview.md)경우 [REST API,](https://aka.ms/ams-v3-rest-ref) [CLI](https://aka.ms/ams-v3-cli-ref)또는 지원되는 [SDK](media-services-apis-overview.md#sdks)중 하나를 사용합니다.

이 문서에서는 Azure Portal을 사용하여 Media Services 계정을 만드는 방법을 보여줍니다.

### <a name="create-a-media-services-account"></a>Media Services 계정 만들기

1. [Azure Portal](https://portal.azure.com/)에 로그인합니다.
1. 리소스**미디어** > **미디어 서비스**만들기 **+를** > 클릭합니다.
1. 미디어 서비스 계정 만들기 섹션에서 필수 값을 **입력합니다.**
    
    | 이름 | 설명 |
    | ---|---|
    |**계정 이름**|새 미디어 서비스 계정의 이름을 입력합니다. Media Services 계정 이름은 공백 없이 모두 소문자로 이루어진 3-24자의 숫자 또는 문자입니다.|
    |**구독**|구독이 두 개 이상인 경우 액세스할 수 있는 Azure 구독 목록에서 구독을 선택합니다.|
    |**리소스 그룹**|새 리소스 또는 기존 리소스를 선택합니다. 리소스 그룹은 수명 주기, 권한 및 정책을 공유하는 리소스의 컬렉션입니다. 자세한 내용은 [여기를 참조하십시오.](../../azure-resource-manager/management/overview.md#resource-groups)|
    |**위치**|Media Services 계정의 미디어 및 메타데이터 레코드를 저장하는 데 사용할 지리적 지역을 선택합니다. 이 지역은 미디어를 처리하고 스트림하는 데 사용됩니다. 사용 가능한 Media Services 지역만 드롭다운 목록 상자에 표시됩니다. |
    |**저장소 계정**|미디어 서비스 계정에서 미디어 콘텐츠의 Blob 저장소를 제공 하려면 저장소 계정을 선택 합니다. Media Services 계정과 동일한 지역의 기존 스토리지 계정을 선택하거나 새 스토리지 계정을 만들 수 있습니다. 동일한 지역에 새 스토리지 계정이 생성됩니다. 스토리지 계정 이름에 대한 규칙은 Media Services 계정의 경우와 같습니다.<br/><br/>**기본** 저장소 계정이 하나 이어야 하며 미디어 서비스 계정과 연결된 **여러 개의 보조** 저장소 계정을 가질 수 있습니다. Azure 포털을 사용하여 보조 저장소 계정을 추가할 수 있습니다. 자세한 내용은 [Azure Media Services 계정이 있는 Azure 저장소 계정을](storage-account-concept.md)참조하십시오.<br/><br/>Media Services 계정 및 연결된 모든 스토리지 계정은 동일한 Azure 구독에 포함되어야 합니다. 추가 대기 시간 및 데이터 송신 비용이 발생하지 않도록 Media Services 계정과 동일한 위치에 있는 스토리지 계정을 사용하는 것이 좋습니다.|
    
1. 계정 배포 진행 상태를 보려면 **대시보드에 고정** 을 선택합니다.
1. 양식 맨 아래에 있는 **만들기** 를 클릭합니다.

    Media Services 계정이 생성되면 **기본** 스트리밍 끝점이 **중지됨** 상태의 계정에 추가됩니다. 콘텐츠 스트리밍을 시작하고 [동적 패키징](dynamic-packaging-overview.md) 및 [동적 암호화를](content-protection-overview.md)활용하려면 콘텐츠를 스트리밍하려는 스트리밍 끝점이 **실행 중** 상태여야 합니다. 

## <a name="use-the-azure-cli"></a>Azure CLI 사용

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

### <a name="set-the-azure-subscription"></a>Azure 구독 설정

다음 명령에서 Media Services 계정에 사용할 Azure 구독 ID를 제공합니다. [구독](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)으로 이동하면 액세스 권한이 있는 구독 목록을 볼 수 있습니다.

```azurecli
az account set --subscription mySubscriptionId
```

### <a name="create-a-resource-group"></a>리소스 그룹 만들기

다음 명령을 사용하여 리소스 그룹을 만듭니다. Azure 리소스 그룹은 Azure Media Services 계정 및 관련 스토리지 계정과 같은 리소스가 배포되어 관리되는 논리적 컨테이너입니다.

당신은 당신의 `amsResourceGroup` 가치로 대체 할 수 있습니다.

```azurecli
az group create --name amsResourceGroup --location westus2
```

### <a name="create-a-storage-account"></a>스토리지 계정 만들기

Media Services 계정을 만들려면 Azure Storage 계정 리소스의 이름을 제공해야 합니다. 지정된 스토리지 계정은 Media Services 계정에 연결됩니다. Media Services에서 스토리지 계정을 사용하는 방법에 대한 자세한 내용은 [스토리지 계정](storage-account-concept.md)을 참조하세요.

**기본** 스토리지 계정이 하나 있어야 하며 미디어 서비스 계정과 연결된 **보조** 스토리지 계정이 여러 개 있을 수 있습니다. Media Services는 **범용 v2**(GPv2) 또는 **범용 v1**(GPv1) 계정을 지원합니다. Blob 전용 계정은 **기본**으로 허용되지 않습니다. 스토리지 계정에 대한 자세한 내용은 [Azure Storage 계정 옵션](../../storage/common/storage-account-options.md)을 참조하세요. 

다음 예제에서는 범용 v2, 표준 LRS 계정을 만듭니다. 스토리지 계정을 실험하려면 `--sku Standard_LRS`를 사용합니다. 그러나 프로덕션용 SKU를 선택하는 경우 비즈니스 연속성을 위해 지리적 복제를 제공하는 `--sku Standard_RAGRS`를 고려해야 합니다. 자세한 내용은 [스토리지 계정](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest)을 참조하세요.
 
다음 명령은 Media Services 계정과 연결될 스토리지 계정을 만듭니다. 아래 스크립트에서 `storageaccountforams`를 원하는 값으로 대체할 수 있습니다. `amsResourceGroup`이전 단계에서 리소스 그룹에 대해 제공한 값과 일치해야 합니다. 저장소 계정 이름의 길이가 24미만이어야 합니다.

```azurecli
az storage account create --name storageaccountforams \  
  --kind StorageV2 \
  --sku Standard_LRS \
  -l westus2 \
  -g amsResourceGroup
```

### <a name="create-a-media-services-account"></a>Media Services 계정 만들기

다음 Azure CLI 명령은 새 Media Services 계정을 만듭니다. 다음 값을 바꿀 수 `amsaccount` `storageaccountforams` 있습니다(저장소 계정에 대해 제공한 값과 `amsResourceGroup` 일치해야 함) 및(리소스 그룹에 대해 제공한 값과 일치해야 함).  

```azurecli
az ams account create --name amsaccount \
   -g amsResourceGroup --storage-account storageaccountforams \
   -l westus2 
```

### <a name="see-also"></a>참조

* [Azure CLI](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
* [미디어 서비스 계정에 보조 저장소 연결](https://docs.microsoft.com/cli/azure/ams/account/storage?view=azure-cli-latest#az-ams-account-storage-add)

---

## <a name="next-steps"></a>다음 단계

[파일 스트리밍](stream-files-dotnet-quickstart.md)
