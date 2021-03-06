---
title: Azure 이벤트 허브에 대한 네트워크 보안
description: 이 문서에서는 개인 끝점에서 액세스를 구성하는 방법을 설명합니다.
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: spelluru
ms.openlocfilehash: 46e6a9ecc2ed09aed1076f12c1f61a966485bdad
ms.sourcegitcommit: 7581df526837b1484de136cf6ae1560c21bf7e73
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2020
ms.locfileid: "80422778"
---
# <a name="network-security-for-azure-event-hubs"></a>Azure 이벤트 허브에 대한 네트워크 보안 
이 문서에서는 Azure Event Hubs에서 다음 보안 기능을 사용하는 방법에 대해 설명합니다. 

- 서비스 태그
- IP 방화벽 규칙
- 네트워크 서비스 끝점
- 비공개 끝점(미리 보기)


## <a name="service-tags"></a>서비스 태그
서비스 태그는 지정된 Azure 서비스의 IP 주소 접두사 그룹을 나타냅니다. Microsoft는 서비스 태그로 둘러싸인 주소 접두사를 관리하고 주소가 변경될 때 서비스 태그를 자동으로 업데이트하여 네트워크 보안 규칙에 대한 자주 업데이트하는 복잡성을 최소화합니다. 서비스 태그에 대한 자세한 내용은 [서비스 태그 개요를](../virtual-network/service-tags-overview.md)참조하십시오.

서비스 태그를 사용하여 [네트워크 보안 그룹](../virtual-network/security-overview.md#security-rules) 또는 Azure [방화벽](../firewall/service-tags.md)에서 네트워크 액세스 제어를 정의할 수 있습니다. 보안 규칙을 만들 때 특정 IP 주소 대신 서비스 태그를 사용합니다. 규칙의 적절한 *소스* 또는 *대상* 필드에 서비스 태그 이름(예: **EventHub)을**지정하면 해당 서비스에 대한 트래픽을 허용하거나 거부할 수 있습니다.

| 서비스 태그 | 목적 | 인바운드 또는 아웃바운드를 사용할 수 있습니까? | 지역일 수 있습니까? | Azure 방화벽에서 사용할 수 있습니까? |
| --- | -------- |:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **이벤트 허브** | Azure 이벤트 허브입니다. | 아웃바운드 | 예 | 예 |


## <a name="ip-firewall"></a>IP 방화벽 
기본적으로 Event Hubs 네임스페이스는 유효한 인증 및 권한 부여와 함께 요청이 제공되는 한 인터넷에서 액세스할 수 있습니다. IP 방화벽을 사용하면 [CIDR(클래스리스 도메인 간 라우팅)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 표기법의 IPv4 주소 또는 IPv4 주소 범위 집합으로만 제한할 수 있습니다.

이 기능은 Azure Event Hubs가 잘 알려진 특정 사이트에서만 액세스할 수 있어야 하는 시나리오에서 유용합니다. 방화벽 규칙을 사용하면 특정 IPv4 주소에서 발생한 트래픽을 허용하도록 규칙을 구성할 수 있습니다. 예를 들어 [Azure Express Route에서](/azure/expressroute/expressroute-faqs#supported-services)이벤트 허브를 사용하는 경우 온-프레미스 인프라 IP 주소에서만 트래픽을 허용하는 **방화벽 규칙을** 만들 수 있습니다. 

IP 방화벽 규칙은 이벤트 허브 네임스페이스 수준에서 적용됩니다. 따라서 해당 규칙은 지원되는 모든 프로토콜을 사용하는 클라이언트의 모든 연결에 적용됩니다. Event Hubs 네임스페이스에서 허용된 IP 규칙과 일치하지 않는 IP 주소의 연결 시도는 권한이 없는 것으로 거부됩니다. 응답은 IP 규칙을 언급하지 않습니다. IP 필터 규칙은 순서대로 적용되며 IP 주소와 일치하는 첫 번째 규칙이 수락 또는 거부 작업을 결정합니다.

자세한 내용은 [이벤트 허브에 대한 IP 방화벽을 구성하는 방법을](event-hubs-ip-filtering.md) 참조하세요.

## <a name="network-service-endpoints"></a>네트워크 서비스 끝점
[VNet(Virtual Network) 서비스 엔드포인트](../virtual-network/virtual-network-service-endpoints-overview.md)와 Event Hubs를 통합하면 양쪽 엔드에서 네트워크 트래픽 경로를 보호하여 가상 네트워크에 바인딩된 가상 머신과 같은 워크로드의 메시징 기능에 대한 액세스를 보호할 수 있습니다.

하나 이상의 가상 네트워크 서브넷 서비스 엔드포인트에 바인딩하도록 구성된 각 Event Hubs 네임스페이스는 더 이상 가상 네트워크의 권한이 부여된 서브넷을 제외한 다른 곳에서의 트래픽을 허용하지 않습니다. 가상 네트워크 큐브 뷰에서 Event Hubs 네임스페이스를 서비스 엔드포인트에 바인딩하면 가상 네트워크 서브넷에서 메시징 서비스로 격리된 네트워킹 터널을 구성합니다. 

메시징 서비스 엔드포인트의 관찰 가능한 네트워크 주소가 공용 IP 범위에 있음에도 서브넷에 바인딩된 워크로드와 해당하는 Event Hubs 네임스페이스 간에 격리된 프라이빗 관계가 생성됩니다. 이 동작에는 예외가 있습니다. 기본적으로 서비스 끝점을 사용하도록 설정하면 `denyall` 가상 네트워크와 연결된 [IP 방화벽의](event-hubs-ip-filtering.md) 규칙을 사용할 수 있습니다. IP 방화벽에 특정 IP 주소를 추가하여 Event Hub 공용 끝점에 액세스할 수 있습니다. 

> [!IMPORTANT]
> 가상 네트워크는 Event Hubs의 **표준** 및 **전용** 계층에서 지원되며 **기본** 계층에서는 지원되지 않습니다.

### <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>VNet 통합에서 사용하도록 설정한 고급 보안 시나리오 

단단하고 구획화된 보안이 필요하고 가상 네트워크 서브넷이 구획화된 서비스 간의 세분화를 제공하는 솔루션은 여전히 해당 구획에 상주하는 서비스 간의 통신 경로가 필요합니다.

TCP/IP에서 HTTPS를 수행하는 경로를 비롯한 구획 간의 즉시 IP 경로를 사용하면 네트워크 레이어에서 취약성을 악용할 위험이 있습니다. 메시징 서비스는 격리된 통신 경로를 제공하며, 여기서 메시지는 당사자 간에 전환할 때 디스크에 기록됩니다. 동일한 Event Hubs 인스턴스에 바인딩된 두 개의 고유 가상 네트워크에 있는 워크로드는 메시지를 통해 효율적이고 안정적으로 통신할 수 있습니다. 이 동안에 해당 네트워크 격리 경계의 무결성은 지켜집니다.
 
즉, 보안이 중요한 클라우드 솔루션은 Azure에서 제공하는 업계 최고의 안정적이고 확장성 있는 비동기 메시징 기능에 액세스할 뿐만 아니라 메시징을 사용하여 HTTPS 및 기타 TLS 보안 소켓 프로토콜을 비롯한 보안 솔루션 구획(피어 간 통신 모드로 달성 가능한 수준보다 본질적으로 더 안전함) 간에 통신을 만들 수도 있습니다.

### <a name="bind-event-hubs-to-virtual-networks"></a>이벤트 허브를 가상 네트워크에 바인딩

**가상 네트워크 규칙**은 Azure Event Hubs 네임스페이스가 특정 가상 네트워크 서브넷의 연결을 수락할지 여부를 제어하는 방화벽 보안 기능입니다.

Virtual Networks에 Event Hubs를 바인딩하는 작업은 2단계 프로세스입니다. 먼저 가상 네트워크의 서브넷에 **가상 네트워크 서비스 끝점을** 만들고 [서비스 끝점 개요](../virtual-network/virtual-network-service-endpoints-overview.md) 문서에 설명된 대로 **Microsoft.EventHub에** 사용하도록 설정해야 합니다. 서비스 엔드포인트를 추가했다면 여기에 **가상 네트워크 규칙**을 사용하여 Event Hubs 네임스페이스를 바인딩합니다.

가상 네트워크 규칙은 가상 네트워크 서브넷을 사용하는 Event Hubs 네임스페이스의 연결입니다. 규칙이 있는 한 서브넷에 바인딩된 모든 워크로드는 Event Hubs 네임스페이스에 대한 액세스 권한이 부여됩니다. 이벤트 허브 자체는 아웃바운드 연결을 설정하지 않으며, 액세스 권한을 얻을 필요가 없으므로 이 규칙을 사용하도록 설정하여 서브넷에 대한 액세스 권한을 부여하지 않습니다.

자세한 내용은 [이벤트 허브에 대한 가상 네트워크 서비스 끝점을 구성하는 방법을](event-hubs-service-endpoints.md) 참조하세요.

## <a name="private-endpoints"></a>프라이빗 엔드포인트

[Azure 개인 링크 서비스를](../private-link/private-link-overview.md) 사용하면 가상 네트워크의 **개인 끝점을** 통해 Azure 서비스(예: Azure 이벤트 허브, Azure Storage 및 Azure Cosmos DB) 및 Azure 호스팅 고객/파트너 서비스에 액세스할 수 있습니다.

개인 끝점은 Azure 개인 링크에서 제공하는 서비스에 개인적으로 안전하게 연결하는 네트워크 인터페이스입니다. 프라이빗 엔드포인트는 VNet의 개인 IP 주소를 사용하여 서비스를 VNet으로 효과적으로 가져옵니다. 서비스에 대한 모든 트래픽은 프라이빗 엔드포인트를 통해 라우팅할 수 있으므로 게이트웨이, NAT 디바이스, ExpressRoute 또는 VPN 연결 또는 공용 IP 주소가 필요하지 않습니다. 가상 네트워크와 서비스 간의 트래픽은 Microsoft 백본 네트워크를 통해 이동하여 공용 인터넷에서 노출을 제거합니다. Azure 리소스의 인스턴스에 연결하여 액세스 제어에서 가장 높은 수준의 세분성을 제공할 수 있습니다.

> [!NOTE]
> 이 기능은 **전용** 계층에서만 지원됩니다. 전용 계층에 대한 자세한 내용은 [이벤트 허브 전용 개요를](event-hubs-dedicated-overview.md)참조하십시오. 
>
> 이 기능은 현재 **미리 보기**상태입니다. 


자세한 내용은 [이벤트 허브에 대한 개인 끝점을 구성하는 방법을](private-link-service.md) 참조하세요.


## <a name="next-steps"></a>다음 단계
다음 문서를 참조하세요.

- [이벤트 허브에 대한 IP 방화벽을 구성하는 방법](event-hubs-ip-filtering.md)
- [이벤트 허브에 대한 가상 네트워크 서비스 끝점을 구성하는 방법](event-hubs-service-endpoints.md)
- [이벤트 허브에 대한 개인 끝점을 구성하는 방법](private-link-service.md)