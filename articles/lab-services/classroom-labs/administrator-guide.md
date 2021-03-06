---
title: Azure 랩 서비스 - 관리자 가이드 | 마이크로 소프트 문서
description: 이 가이드는 Azure Lab 서비스를 사용하여 랩 계정을 만들고 관리하는 관리자에게 도움이 됩니다.
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/19/2019
ms.author: spelluru
ms.openlocfilehash: 8608aaab7bb8b6d10e67f27678c17f20a6c243da
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80370847"
---
# <a name="azure-lab-services---administrator-guide"></a>Azure 랩 서비스 - 관리자 가이드
대학의 클라우드 리소스를 관리하는 IT(정보 기술) 관리자는 일반적으로 학교에 대한 랩 계정을 설정해야 합니다. 랩 계정이 설정되면 관리자 또는 교육자는 랩 계정에 포함된 강의실 랩을 만듭니다. 이 문서에서는 관련된 Azure 리소스에 대한 자세한 개요와 이를 만들기 위한 지침을 제공합니다.

![랩 계정의 Azure 리소스에 대한 상위 수준 보기](../media/administrator-guide/high-level-view.png)

- 클래스룸 랩은 Azure 랩 서비스에서 소유한 Azure 구독 내에서 호스팅됩니다.
- 랩 계정, 공유 이미지 갤러리 및 이미지 버전은 구독 내에서 호스팅됩니다.
- 랩 계정과 공유 이미지 갤러리를 동일한 리소스 그룹에 가질 수 있습니다. 이 다이어그램에서는 서로 다른 리소스 그룹에 있습니다. 

## <a name="subscription"></a>Subscription
대학에 하나 이상의 Azure 구독이 있습니다. 구독은 랩 계정을 포함하여 그 안에 사용되는 모든 Azure 리소스\서비스에 대한 청구 및 보안을 관리하는 데 사용됩니다.

랩 계정과 해당 구독 간의 관계는 다음과 같은 이유로 중요합니다.

- 청구는 랩 계정이 포함된 구독을 통해 보고됩니다.
- 구독의 AD(Azure Active Directory) 테넌트 액세스 권한을 사용자에게 Azure 랩 서비스에 제공할 수 있습니다. 사용자를 랩 계정 소유자\기여자, 강의실 랩 작성자 또는 강의실 랩 소유자로 추가할 수 있습니다.

클래스룸 랩과 해당 VM(가상 머신)은 Azure Lab Services가 소유한 구독 내에서 관리되고 호스팅됩니다.

## <a name="resource-group"></a>Resource group
구독에는 하나 이상의 리소스 그룹이 포함됩니다. 리소스 그룹은 동일한 솔루션 내에서 함께 사용되는 Azure 리소스의 논리적 그룹을 만드는 데 사용됩니다.  

랩 계정을 만들 때 랩 계정이 포함된 리소스 그룹을 구성해야 합니다. 

[공유 이미지 갤러리를](#shared-image-gallery)만들 때 리소스 그룹도 필요합니다. 랩 계정과 공유 이미지 갤러리를 두 개의 별도 리소스 그룹에 배치하도록 선택할 수 있습니다. 또는 동일한 리소스 그룹에 배치하도록 선택할 수 있습니다.

랩 계정을 만들 때 공유 이미지 갤러리를 동시에 자동으로 만들고 첨부할 수 있습니다.  이 옵션을 사용하면 랩 계정과 공유 이미지 갤러리가 별도의 리소스 그룹에서 만들어집니다. 이 자습서에서 설명한 단계를 사용 하는 경우이 동작을 볼 수 [있습니다.](how-to-attach-detach-shared-image-gallery.md#configure-at-the-time-of-lab-account-creation) 이 문서의 맨 위에 있는 이미지도 이 구성을 사용합니다. 

랩 계정 또는 공유 이미지 갤러리의 리소스 그룹을 만든 후에는 변경할 수 *없으므로* 리소스 그룹의 구조를 계획하기 위해 미리 시간을 투자하는 것이 좋습니다. 이러한 리소스에 대한 리소스 그룹을 변경해야 하는 경우 랩 계정 및\또는 공유 이미지 갤러리를 삭제하고 다시 만들어야 합니다.

## <a name="lab-account"></a>랩 계정
랩 계정은 하나 이상의 강의실 랩에 대한 컨테이너 역할을 합니다. Azure Lab 서비스를 시작할 때는 단일 랩 계정만 있는 것이 일반적입니다. 랩 사용량이 확장되면 나중에 더 많은 랩 계정을 만들도록 선택할 수 있습니다.

다음 목록에서는 두 개 이상의 랩 계정이 도움이 될 수 있는 시나리오를 강조 합니다.

- **교실 랩 전반에서 다양한 정책 요구 사항 관리** 
    
    랩 계정을 설정할 때 다음과 같이 랩 계정아래의 *모든* 강의실 랩에 적용되는 정책을 설정합니다.
    - 클래스룸 랩에서 액세스할 수 있는 공유 리소스가 있는 Azure 가상 네트워크입니다. 예를 들어 가상 네트워크 내에서 공유 데이터 집합에 액세스해야 하는 클래스룸 랩 집합이 있을 수 있습니다.
    - 교실 랩에서 VM을 만드는 데 사용할 수 있는 VM(가상 컴퓨터) 이미지입니다. 예를 들어 Linux 마켓플레이스 이미지용 [데이터 과학 VM에](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.linux-data-science-vm-ubuntu) 액세스해야 하는 일련의 교실 랩이 있을 수 있습니다. 
    
    서로 고유한 정책 요구 사항이 있는 강의실 랩이 있는 경우 이러한 강의실 랩을 별도로 관리하기 위해 별도의 랩 계정을 만드는 것이 좋습니다.

- **랩 계정별로 별도의 예산**
  
    단일 랩 계정을 통해 모든 강의실 랩 비용을 보고하는 대신 보다 명확하게 구분된 예산이 필요할 수 있습니다. 예를 들어 대학의 수학 부서, 컴퓨터 과학 부서 등에 대한 랩 계정을 만들어 부서 간에 예산을 구분할 수 있습니다.  그런 다음 [Azure 비용 관리를](https://docs.microsoft.com/azure/cost-management-billing/cost-management-billing-overview)사용하여 각 개별 랩 계정의 비용을 볼 수 있습니다.
    
- **활성\생산 실험실에서 파일럿 실험실 격리**
  
    active\production 랩에 영향을 주지 않고 랩 계정에 대한 정책 변경 을 시험 운용하려는 경우가 있을 수 있습니다. 이 유형의 시나리오에서는 시험 운용을 위해 별도의 랩 계정을 만들면 변경 내용을 격리할 수 있습니다. 

## <a name="classroom-lab"></a>교실 실습실
교실 랩에는 각각 단일 학생에게 할당된 가상 컴퓨터(VM)가 포함되어 있습니다. 일반적으로 다음을 기대할 수 있습니다.

- 각 수업마다 하나의 교실 랩을 갖습니다.
- 매 학기 마다 새로운 교실 랩 세트를 만듭니다(또는 수업이 제공되는 각 기간마다). 일반적으로 동일한 이미지가 필요한 클래스의 경우 [공유 이미지 갤러리를](#shared-image-gallery) 사용하여 랩 및 학기 간 이미지를 재사용해야 합니다.

교실 랩을 구성하는 방법을 결정할 때 다음 사항을 고려하십시오.

- **교실 랩 내의 모든 VM은 게시된 동일한 이미지로 배포됩니다.**

    따라서 다른 랩 이미지를 동시에 게시해야 하는 클래스가 있는 경우 각 랩에 대해 별도의 강의실 랩을 만들어야 합니다.
  
- **사용 할당량은 랩 수준에서 설정되며 랩 내의 모든 사용자에게 적용됩니다.**
    
    사용자에 대해 다른 할당량을 설정하려면 별도의 강의실 랩을 만들어야 합니다. 그러나 할당량을 설정한 후 특정 사용자에게 시간을 더 추가할 수 있습니다.
  
- **시작 또는 종료 일정은 랩 수준에서 설정되며 랩 내의 모든 VM에 적용됩니다.**

    이전 지점과 마찬가지로 사용자에 대해 다른 일정을 설정해야 하는 경우 별도의 강의실 랩을 만들어야 합니다. 

## <a name="shared-image-gallery"></a>공유 이미지 갤러리
공유 이미지 갤러리는 랩 계정에 첨부되며 이미지를 저장하기 위한 중앙 리포지토리 역할을 합니다. 교육자가 교실 랩의 템플릿 가상 시스템(VM)에서 내보내기를 선택하면 이미지가 갤러리에 저장됩니다. 교육자가 템플릿 VM및 내보내기를 변경할 때마다 이전 버전을 유지하면서 새 버전의 이미지가 저장됩니다.

교수자는 새 강의실 랩을 만들 때 공유 이미지 갤러리에서 이미지 버전을 게시할 수 있습니다. 갤러리는 여러 버전의 이미지를 저장할 수 있지만 교육자는 랩을 만드는 동안에만 최신 버전을 선택할 수 있습니다.

공유 이미지 갤러리는 몇 개의 강의실 랩으로 시작할 때 즉시 필요하지 않을 수도 있는 선택적 리소스입니다. 그러나 공유 이미지 갤러리를 사용하면 더 많은 강의실 랩을 사용할 때 유용한 많은 이점이 있습니다.

- **템플릿 VM 이미지의 버전을 저장하고 관리할 수 있습니다.**

    사용자 지정 이미지를 만들거나 공용 마켓플레이스 갤러리의 이미지(소프트웨어, 구성 등)를 변경하는 것이 유용합니다.  예를 들어 교육자는 다른 소프트웨어\툴링을 설치하도록 요구하는 것이 일반적입니다. 학생이 이러한 필수 구성 조건을 직접 설치하도록 요구하는 대신 다양한 버전의 템플릿 VM 이미지를 공유 이미지 갤러리로 내보낼 수 있습니다. 그런 다음 새 강의실 랩을 만들 때 이러한 이미지 버전을 사용할 수 있습니다.
- **교실 랩 에서 템플릿 VM 이미지 공유\재사용 가능**

    새 교실 랩을 만들 때마다 이미지를 처음부터 구성할 필요가 없도록 이미지를 저장하고 다시 사용할 수 있습니다. 예를 들어 동일한 이미지가 필요한 여러 클래스가 제공되는 경우 이 이미지를 한 번만 만들고 공유 이미지 갤러리로 내보내서 교실 랩 에서 공유할 수 있습니다.
- **복제를 통해 이미지 가용성 보장**

    교실 실습실에서 공유 이미지 갤러리에 저장하면 이미지가 [동일한 지역 내의](https://azure.microsoft.com/global-infrastructure/regions/)다른 지역으로 자동으로 복제됩니다. 지역에 대한 중단이 있는 경우 다른 지역의 이미지 복제본을 사용할 수 있으므로 이미지를 교실 랩에 게시하는 것은 영향을 받지 않습니다.  여러 복제본에서 VM을 게시하면 성능에도 도움이 될 수 있습니다.

공유 이미지를 논리적으로 그룹화하려면 다음과 같은 몇 가지 옵션이 있습니다.

- 여러 공유 이미지 갤러리를 만듭니다. 각 랩 계정은 하나의 공유 이미지 갤러리에만 연결할 수 있으므로 이 옵션을 사용하려면 여러 랩 계정을 만들어야 합니다.
- 또는 여러 랩 계정에서 공유하는 단일 공유 이미지 갤러리를 사용할 수 있습니다. 이 경우 각 랩 계정은 포함된 강의실 랩에 적용할 수 있는 이미지만 사용하도록 설정할 수 있습니다.

## <a name="naming"></a>이름 지정
Azure Lab Services를 시작할 때 리소스 그룹, 랩 계정, 강의실 랩 및 공유 이미지 갤러리에 대한 명명 규칙을 설정하는 것이 좋습니다. 설정한 명명 규칙은 조직의 요구 사항에 고유하지만 다음 표에는 일반적인 지침이 요약되어 있습니다.

| 리소스 유형 | 역할 | 제안된 패턴 | 예 |
| ------------- | ---- | ----------------- | -------- | 
| Resource group | 하나 이상의 랩 계정과 하나 이상의 공유 이미지 갤러리포함 | \<조직 짧은\>-\<\>이름 환경 -rg<ul><li>**조직 짧은 이름은** 리소스 그룹이 지원하는 조직의 이름을 식별합니다.</li><li>**환경은** 파일럿 또는 프로덕션과 같은 리소스에 대한 환경을 식별합니다.</li><li>**Rg는** 리소스 유형인 리소스 그룹을 의미합니다.</li></ul> | 콘토소유니더랩-rg<br/>콘토소유니드랩스-파일럿-rg<br/>콘토소유니더랩스-프로드-rg |
| 랩 계정 | 하나 이상의 랩을 포함합니다. | \<조직 짧은\>-\<\>이름 환경 -la<ul><li>**조직 짧은 이름은** 리소스 그룹이 지원하는 조직의 이름을 식별합니다.</li><li>**환경은** 파일럿 또는 프로덕션과 같은 리소스에 대한 환경을 식별합니다.</li><li>**La는** 리소스 유형인 랩 계정을 의미합니다.</li></ul> | 콘토소유니더랩스 라<br/>마학슬랩 라<br/>사이언스디브랩파일럿-라<br/>사이언스디브랩스 프로드 라 |
| 교실 실습실 | 하나 이상의 VM을 포함합니다. |\<클래스\>-\<이름\>-\<기간 교육자 식별자\><ul><li>**클래스 이름은** 랩이 지원하는 클래스의 이름을 식별합니다.</li><li>**기간은** 클래스가 제공되는 기간을 식별합니다.</li>**교육 식별자는** 랩을 소유한 교육자를 식별합니다.</li></ul> | CS1234-fall2019-johndoe<br/>CS1234-spring2019-johndoe | 
| 공유 이미지 갤러리 | 하나 이상의 VM 이미지 버전포함 | \<조직 짧은\>이름 갤러리 | 콘토소유니더랩스갤러리 |

다른 Azure 리소스의 이름을 지정하는 방법에 대한 자세한 내용은 [Azure 리소스에 대한 명명 규칙을](/azure/architecture/best-practices/naming-conventions)참조하십시오.

## <a name="regionslocations"></a>지역\위치

Azure Lab Services의 리소스를 설정할 때 리소스를 호스트할 데이터 센터의 지역(또는 위치)을 제공해야 합니다. 다음은 랩 설정과 관련된 각 리소스에 대한 지역 영향방법에 대한 자세한 내용입니다.

### <a name="resource-group"></a>Resource group

이 지역은 리소스 그룹에 대한 정보가 저장되는 데이터 센터를 지정합니다. 리소스 그룹에 포함된 Azure 리소스는 상위 리소스와 다른 지역에 있을 수 있습니다.

### <a name="lab-account"></a>랩 계정

랩 계정의 위치는 이 리소스가 있는 지역을 나타냅니다.  

### <a name="classroom-lab"></a>교실 실습실
    
교실 랩이 있는 위치는 다음 요인에 따라 다릅니다.

  - **랩 계정이 가상 네트워크(VNet)로 피어**
  
    랩 계정은 동일한 지역에 있을 때 [VNet으로 피어볼](https://docs.microsoft.com/azure/lab-services/classroom-labs/how-to-connect-peer-virtual-network) 수 있습니다.  랩 계정이 VNet으로 피어면 랩 계정 및 VNet과 동일한 지역에서 강의실 랩이 자동으로 만들어집니다.

    > [!NOTE]
    > 랩 계정이 VNet을 사용하여 피어링되면 **랩 작성자가 랩 위치를 선택할 수 있도록 허용하는** 설정이 비활성화됩니다. 이 설정에 대한 추가 정보는 이 문서에서 찾을 수 [있습니다.](https://docs.microsoft.com/azure/lab-services/classroom-labs/allow-lab-creator-pick-lab-location)
    
  - **VNet이 피어링되지 ***않으며*** 랩 작성자가 랩 위치를 선택할 수 없습니다.**
  
    랩 *계정을* 피어링한 VNet이 **없고** [랩 작성자가 랩 위치를 선택할 수 **없는** ](https://docs.microsoft.com/azure/lab-services/classroom-labs/allow-lab-creator-pick-lab-location)경우 사용 가능한 VM 용량이 있는 리전에서 강의실 랩이 자동으로 만들어집니다.  특히 Azure Lab 서비스는 랩 [계정과 동일한 지역 내에 있는 리전에서 가용성을](https://azure.microsoft.com/global-infrastructure/regions)찾습니다.

  - **VNet이 피어링되지 ***않으며*** 랩 작성자가 랩 위치를 선택할 수 있습니다.**
       
    VNet 피어링이 **없고** [랩 작성자가 랩 위치를 선택할 수](https://docs.microsoft.com/azure/lab-services/classroom-labs/allow-lab-creator-pick-lab-location)있는 경우 랩 작성자가 선택할 수 있는 위치는 사용 가능한 용량을 기반으로 합니다.

일반적인 규칙은 리소스의 지역을 사용자에게 가장 가까운 지역으로 설정하는 것입니다. 이는 강의실 랩의 경우 학생과 가장 가까운 교실 랩을 만드는 것을 의미합니다. 전 세계에 위치한 온라인 코스의 경우 최선의 판단을 통해 중앙에 위치한 교실 랩을 만들어야 합니다. 또는 학생의 지역에 따라 수업을 여러 강의실 랩으로 분할합니다.

### <a name="shared-image-gallery"></a>공유 이미지 갤러리

이 영역은 첫 번째 이미지 버전이 대상 영역에 자동으로 복제되기 전에 저장되는 소스 영역을 나타냅니다.

## <a name="vm-sizing"></a>VM 크기 조정
관리자 나 랩 작성자가 강의실 랩을 만들 때 강의실의 필요에 따라 다음 VM 크기 중에서 선택할 수 있습니다. 사용 가능한 계산 크기는 랩 계정이 있는 지역에 따라 다릅니다.

| 크기 | 사양 | 계열 | 권장 사용 |
| ---- | ----- | ------ | ------------- |
| 작음| <ul><li>2개 코어</li><li>3.5GB RAM</li> | [Standard_A2_v2](https://docs.microsoft.com/azure/virtual-machines/av2-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) | 이 크기는 명령줄, 웹 브라우저 열기, 트래픽이 적은 웹 서버, 중소 규모 데이터베이스에 가장 적합합니다. |
| 중간 | <ul><li>4 코어</li><li>7GB RAM</li> | [Standard_A4_v2](https://docs.microsoft.com/azure/virtual-machines/av2-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) | 이 크기는 관계형 데이터베이스, 메모리 내 캐싱 및 분석에 가장 적합합니다. |
| 중간(중첩 가상화) | <ul><li>4 코어</li><li>16GB RAM</li></ul> | [Standard_DC4s_v2](https://docs.microsoft.com/azure/virtual-machines/dcv2-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) | 이 크기는 관계형 데이터베이스, 메모리 내 캐싱 및 분석에 가장 적합합니다.  이 크기는 중첩된 가상화도 지원합니다. |
| 큰 | <ul><li>8개 코어</li><li>32GB RAM</li></ul>  | [Standard_DC8_v2](https://docs.microsoft.com/azure/virtual-machines/dcv2-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json) | 이 크기는 더 빠른 CPU, 더 나은 로컬 디스크 성능, 대규모 데이터베이스, 대용량 메모리 캐시가 필요한 응용 프로그램에 가장 적합합니다.  이 크기는 중첩된 가상화도 지원합니다. |
| 소형 GPU(시각화) | <ul><li>6 코어</li><li>56GB RAM</li>  | [Standard_NV6](https://docs.microsoft.com/azure/virtual-machines/nv-series) | 이 크기는 OpenGL 및 DirectX와 같은 프레임워크를 사용하여 원격 시각화, 스트리밍, 게임, 인코딩에 가장 적합합니다. |
| 소형 GPU(컴퓨팅) | <ul><li>6 코어</li><li>56GB RAM</li></ul>  | [Standard_NC6](https://docs.microsoft.com/azure/virtual-machines/nc-series) |이 크기는 인공 지능 및 딥 러닝과 같은 컴퓨터 집약적 응용 프로그램에 가장 적합합니다. |
| 중간 GPU(시각화) | <ul><li>12 코어</li><li>112GB RAM</li></ul>  | [Standard_NC12](https://docs.microsoft.com/azure/virtual-machines/nc-series) | 이 크기는 OpenGL 및 DirectX와 같은 프레임워크를 사용하여 원격 시각화, 스트리밍, 게임, 인코딩에 가장 적합합니다. |

## <a name="manage-identity"></a>ID 관리
[Azure의 역할 기반 액세스 제어를](https://docs.microsoft.com/azure/role-based-access-control/overview)사용하여 다음 역할을 할당하여 랩 계정 및 강의실 랩에 대한 액세스 권한을 부여할 수 있습니다.

- **랩 계정 소유자**

    랩 계정을 만드는 관리자는 랩 계정의 **소유자** 역할에 자동으로 추가됩니다.  **소유자** 역할이 할당된 관리자는 다음을 수행할 수 있습니다.
     - 랩 계정의 설정을 변경합니다.
     - 다른 관리자에게 랩 계정에 대한 액세스 권한을 소유자 또는 기여자로 부여합니다. 
     - 교육자에게 크리에이터, 소유자 또는 기여자로서 교실 랩에 액세스할 수 있도록 합니다.
     - 랩 계정에서 모든 강의실 랩을 만들고 관리합니다.

- **랩 계정 기여자**

    **기여자** 역할을 할당한 관리자는 다음을 수행할 수 있습니다.
    - 랩 계정의 설정을 변경합니다.
    - 랩 계정 내의 모든 교실 랩을 만들고 관리합니다.
    
    그러나 다른 사용자에게 랩 계정 또는 강의실 랩에 대한 액세스 권한을 *부여할 수는 없습니다.*

- **교실 랩 작성자**

    랩 계정 내에서 강의실 랩을 만들려면 교육자가 **랩 작성자** 역할의 구성원이어야 합니다.  교육자가 교실 랩을 만들면 자동으로 랩의 소유자로 추가됩니다.  [ **랩 작성자** 역할에 사용자를 추가하는](https://docs.microsoft.com/azure/lab-services/classroom-labs/tutorial-setup-lab-account#add-a-user-to-the-lab-creator-role)방법에 대한 자습서를 참조하십시오. 

- **교실 랩 소유자\기고자**
  
    교육자는 랩의 **소유자** 또는 **기여자** 역할의 구성원인 경우 강의실 랩의 설정을 보고 변경할 수 있습니다. 또한 랩 계정의 **Reader** 역할의 구성원이어야 합니다.

    랩의 **소유자** 역할과 **기여자** 역할의 주요 차이점은 기여자가 다른 사용자에게 랩을 관리할 수 있는 액세스 권한을 부여할 *수 없으며* 소유자만 다른 사용자에게 랩을 관리할 수 있는 액세스 권한을 부여할 수 있다는 것입니다.
    
    또한 교육자는 **랩 작성자** 역할의 구성원이 아니면 새 강의실 랩을 만들 *수 없습니다.*

- **공유 이미지 갤러리**
    
    랩 계정에 공유 이미지 갤러리를 연결하면 랩 계정 소유자\기여자 및 랩 작성자\소유자\기여자가 갤러리에서 이미지를 보고 저장할 수 있는 액세스 권한이 자동으로 부여됩니다. 

역할 할당에 도움이 되는 몇 가지 팁은 다음과 같습니다.
   - 일반적으로 관리자는 랩 계정의 **소유자** 또는 **기여자** 역할의 구성원이어야 합니다. 소유자가 두 명 이상일 수 있습니다.

   - 교육자에게 새로운 교실 랩을 만들고 그들이 만드는 랩을 관리할 수 있는 기능을 제공합니다. 랩 크리에이터 역할에 대한 액세스 권한을 할당하기만 하면 **됩니다.**
   
   - 교육자에게 특정 강의실을 관리할 수 있는 기능을 제공하지만 새 랩을 만들 수 있는 기능은 *제공하지 않습니다.* 관리하려는 각 강의실 랩에 대한 **소유자** 또는 **기여자** 역할에 대한 액세스 권한을 할당해야 합니다.  예를 들어 교수와 조교가 모두 교실 랩을 공동 소유하도록 허용할 수 있습니다.  [사용자를 클래스룸 랩에 소유자로 추가하는](https://docs.microsoft.com/azure/lab-services/classroom-labs/how-to-add-user-lab-owner)방법에 대한 가이드를 참조하십시오.

## <a name="pricing"></a>가격 책정

### <a name="azure-lab-services"></a>Azure Lab Services
Azure 랩 서비스의 가격 책정은 다음 [Azure Lab Services pricing](https://azure.microsoft.com/pricing/details/lab-services/)문서에서 설명합니다.

또한 이미지 버전을 저장하고 관리하는 데 사용할 계획이라면 공유 이미지 갤러리의 가격 책정을 고려해야 합니다. 

### <a name="shared-image-gallery"></a>공유 이미지 갤러리
공유 이미지 갤러리를 만들고 랩 계정에 첨부하는 것은 무료입니다. 갤러리에 이미지 버전을 저장할 때까지 비용이 발생하지 않습니다. 일반적으로 공유 이미지 갤러리 사용에 대한 가격은 상당히 무시할 수 있지만 Azure Lab 서비스의 가격 책정에 포함되지 않으므로 계산 방법을 이해하는 것이 중요합니다.  

#### <a name="storage-charges"></a>스토리지 요금
이미지 버전을 저장하기 위해 공유 이미지 갤러리는 표준 HDD 관리 디스크를 사용합니다. 사용되는 HDD 관리 디스크의 크기는 저장되는 이미지 버전의 크기에 따라 다릅니다. 가격 책정을 보려면 다음 문서를 [참조하세요: 관리되는 디스크 가격.](https://azure.microsoft.com/pricing/details/managed-disks/)


#### <a name="replication-and-network-egress-charges"></a>복제 및 네트워크 송신 요금
클래스룸 랩의 템플릿 가상 시스템(VM)을 사용하여 이미지 버전을 저장하는 경우 Azure Lab Services는 먼저 원본 지역에 저장한 다음 소스 이미지 버전을 하나 이상의 대상 영역에 자동으로 복제합니다. Azure Lab Services는 원본 이미지 버전을 클래스룸이 있는 지역 [내의](https://azure.microsoft.com/global-infrastructure/regions/) 모든 대상 지역에 자동으로 복제합니다. 예를 들어 강의실 랩이 미국 지역에 있는 경우 이미지 버전은 미국 내에 있는 8개 지역 각각에 복제됩니다.

이미지 버전이 원본 리전에서 추가 대상 영역으로 복제될 때 네트워크 송신 요금이 발생합니다. 청구된 금액은 이미지 데이터가 처음 원본 영역에서 아웃바운드로 전송될 때 이미지 버전의 크기를 기준으로 합니다.  가격 세부 정보는 다음 문서를 [참조하십시오.](https://azure.microsoft.com/pricing/details/bandwidth/)

[교육 솔루션](https://www.microsoft.com/licensing/licensing-programs/licensing-for-industries?rtc=1&activetab=licensing-for-industries-pivot:primaryr3) 고객은 송신 요금 지불에서 면제될 수 있습니다. 자세한 내용은 계정 관리자에게 문의하세요.  자세한 내용은 링크된 **문서의 FAQ** 섹션, 특히 "학술 고객을 위한 데이터 전송 프로그램이 어떤 경우, 어떻게 자격을 갖추어야 합니까?" 질문을 참조하십시오.

#### <a name="pricing-example"></a>가격 책정 예
위에서 설명한 가격 책정을 요약하면 템플릿 VM 이미지를 공유 이미지 갤러리에 저장하는 예제를 살펴보겠습니다. 다음 시나리오를 가정합니다.

- 사용자 지정 VM 이미지가 하나 있습니다.
- 두 가지 버전의 이미지를 저장합니다.
- 실험실은 총 8개 지역으로 구성된 미국에 있습니다.
- 각 이미지 버전은 크기가 32GB입니다. 그 결과, HDD 관리 디스크 가격은 월 $1.54입니다.

총 비용은 다음과 같이 추정됩니다.

이미지 수 × 버전 수 × 복제본 수 × 관리 디스크 가격

이 예제에서 비용은 다음과 같은 것입니다.

사용자 지정 이미지 1개(32GB) x 2버전 x 8미국 리전 x $1.54 = 월24.64 USD

#### <a name="cost-management"></a>비용 관리
랩 계정 관리자는 갤러리에서 불필요한 이미지 버전을 정기적으로 삭제하여 비용을 관리하는 것이 중요합니다. 

비용을 줄이기 위한 방법으로 특정 지역에 대한 복제를 삭제해서는 안 됩니다(이 옵션은 공유 이미지 갤러리에 있음). 복제 변경은 공유 이미지 갤러리에 저장된 이미지에서 VM을 게시하는 Azure Lab Service의 기능에 부정적인 영향을 미칠 수 있습니다.

## <a name="next-steps"></a>다음 단계
랩 계정 및 랩을 만드는 단계별 지침은 자습서를 [참조하세요.](tutorial-setup-lab-account.md)
