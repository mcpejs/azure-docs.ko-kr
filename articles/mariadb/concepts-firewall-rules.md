---
title: 방화벽 규칙 - MariaDB용 Azure 데이터베이스
description: 방화벽 규칙을 사용하여 MariaDB 서버에 대한 Azure 데이터베이스에 대한 연결을 활성화하는 방법에 대해 알아봅니다.
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 743e3f50d747993250399493d97fc2becab19319
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79532045"
---
# <a name="azure-database-for-mariadb-server-firewall-rules"></a>Azure Database for MariaDB 서버 방화벽 규칙
방화벽은 권한이 있는 컴퓨터를 지정할 때까지 데이터베이스 서버에 대한 모든 액세스를 금지합니다. 방화벽은 각 요청이 시작된 IP 주소의 서버에 대한 액세스를 허용합니다.

방화벽을 구성하려면 허용 가능한 IP 주소 범위를 지정하는 방화벽 규칙을 생성합니다. 서버 수준의 방화벽 규칙을 만들 수 있습니다.

**방화벽 규칙:** 이 규칙은 모든 Azure Database for MariaDB 서버, 즉, 동일한 논리 서버 내의 모든 데이터베이스에 클라이언트가 액세스할 수 있도록 합니다. Azure Portal 또는 Azure CLI 명령을 사용하여 서버 수준 방화벽 규칙을 구성할 수 있습니다. 서버 수준 방화벽 규칙을 만들려면 구독 소유자 또는 구독 참가자여야 합니다.

## <a name="firewall-overview"></a>방화벽 개요
Azure Database for MariaDB 서버에 대한 모든 데이터베이스 액세스는 방화벽에 의해 차단됩니다. 다른 컴퓨터에서 서버를 사용하려면 해당 서버에 대한 액세스를 허용하는 하나 이상의 서버 수준 방화벽 규칙을 지정해야 합니다. 방화벽 규칙을 사용하여 허용할 인터넷에서의 IP 주소 범위를 지정합니다. Azure Portal 웹 사이트 자체에 대한 액세스는 이 방화벽 규칙의 영향을 받지 않습니다.

인터넷과 Azure로부터의 연결 시도는 다음 다이어그램과 같이 Azure Database for MariaDB 데이터베이스에 연결하기 전에 먼저 방화벽을 통과해야 합니다.

![방화벽 작동 방식을 보여 주는 예제 흐름](./media/concepts-firewall-rules/1-firewall-concept.png)

## <a name="connecting-from-the-internet"></a>인터넷에서 연결하기
서버 수준 방화벽 규칙은 Azure Database for MariaDB 서버의 모든 데이터베이스에 적용됩니다.

요청된 IP 주소가 서버 수준 방화벽 규칙의 지정된 범위 안에 있으면 연결이 허용됩니다.

요청 IP 주소가 데이터베이스 수준 또는 서버 수준 방화벽 규칙의 지정된 범위 밖에 있으면 연결 요청이 실패합니다.

## <a name="connecting-from-azure"></a>Azure에서 연결
응용 프로그램 이나 서비스의 나가는 IP 주소를 찾아 명시적으로 해당 개별 IP 주소 또는 범위에 대 한 액세스를 허용 하는 것이 좋습니다. 예를 들어 Azure App Service의 나가는 IP 주소를 찾거나 가상 시스템 또는 기타 리소스에 연결된 공용 IP를 사용할 수 있습니다(서비스 끝점을 통해 가상 시스템의 개인 IP와 연결하는 방법에 대한 정보는 아래 참조). 

Azure 서비스에 대해 고정된 발신 IP 주소를 사용할 수 없는 경우 모든 Azure 데이터 센터 IP 주소에서 연결을 사용하도록 설정하는 것이 좋습니다. Azure **서비스에 대한 액세스 허용** 옵션을 연결 **보안** 창에서 **ON으로** 설정하고 **저장**을 누르면 Azure 포털에서 이 설정을 사용할 수 있습니다. Azure CLI에서 시작 및 종료 주소가 0.0.0.0과 같은 방화벽 규칙 설정은 해당 작업을 수행합니다. 연결 시도가 허용되지 않으면 해당 요청이 Azure Database for MariaDB 서버에 도달하지 않습니다.

> [!IMPORTANT]
> **Azure 서비스 에 대한 액세스 허용** 옵션은 다른 고객의 구독연결을 포함하여 Azure의 모든 연결을 허용하도록 방화벽을 구성합니다. 이 옵션을 선택할 때 로그인 및 사용자 권한이 부여된 사용자만으로 액세스를 제한하는지 확인합니다.
> 

![포털에서 Azure 서비스 방문 허용 구성](./media/concepts-firewall-rules/allow-azure-services.png)

### <a name="connecting-from-a-vnet"></a>VNet에서 연결
VNet에서 MariaDB 서버에 대한 Azure 데이터베이스에 안전하게 연결하려면 [VNet 서비스 끝점을](./concepts-data-access-security-vnet.md)사용하는 것이 좋습니다. 

## <a name="programmatically-managing-firewall-rules"></a>방화벽 규칙을 프로그래밍 방식으로 관리
Azure Portal 외에도 Azure CLI를 사용하여 방화벽 규칙을 프로그래밍 방식으로 관리할 수 있습니다. 

또한 [Azure CLI를 사용하여 MariaDB 방화벽 규칙에 대한 Azure 데이터베이스 만들기 및 관리를](./howto-manage-firewall-cli.md)참조하십시오.

## <a name="troubleshooting-firewall-issues"></a>방화벽 문제 해결
Microsoft Azure Database for MariaDB 서버 서비스에 대한 액세스가 예상대로 작동되지 않는 경우 다음 사항을 고려하세요.

* **허용 목록의 변경사항이 아직 적용되지 않았습니다.** Azure Database for MariaDB 서버 방화벽 구성에 변경 내용이 적용되려면 최대 5분 정도 걸릴 수 있습니다.

* **로그인이 올바르지 않거나 암호가 올바르지 않습니다.** 로그인에 Azure Database for MariaDB 서버에 대한 권한이 없거나 사용한 암호가 틀렸을 경우 Azure Database for MariaDB 서버에 대한 연결이 거부됩니다. 방화벽 설정은 클라이언트에게 서버에 연결을 시도할 수 있는 기회를 제공합니다. 각 클라이언트는 꼭 필요한 보안 자격 증명을 제공해야 합니다.

* **동적 IP 주소:** 동적 IP 주소 지정을 통해 인터넷에 연결되어 있고 방화벽을 통과하는 데 문제가 있는 경우 다음 솔루션 중 하나를 시도할 수 있습니다.

   * ISP(인터넷 서비스 공급자)는 Azure Database for MariaDB 서버에 연결될 클라이언트에 할당된 IP 주소 범위를 요청하고, 방화벽 규칙에 따라 IP 주소 범위를 추가합니다.

   * 클라이언트 컴퓨터 대신 고정 IP 주소를 얻고, 방화벽 규칙에 따라 IP 주소 범위를 추가합니다.

* **서버의 IP가 공용으로 나타납니다.** MariaDB 서버에 대한 Azure 데이터베이스에 대한 연결은 공개적으로 액세스할 수 있는 Azure 게이트웨이를 통해 라우팅됩니다. 그러나 실제 서버 IP는 방화벽으로 보호됩니다. 자세한 내용은 연결 [아키텍처 문서를](concepts-connectivity-architecture.md)참조하십시오. 

## <a name="next-steps"></a>다음 단계
- [Azure Portal을 사용한 Azure Database for MariaDB 방화벽 규칙 만들기 및 관리](./howto-manage-firewall-portal.md)
- [Azure CLI를 사용하여 Azure Database for MariaDB 방화벽 규칙 만들기 및 관리](./howto-manage-firewall-cli.md)
- [MariaDB에 대 한 Azure 데이터베이스에서 VNet 서비스 끝점](./concepts-data-access-security-vnet.md)
