---
title: Azure VM에서 SQL 서버 백업에 대한 Azure 백업 지원 매트릭스
description: Azure 백업 서비스를 통해 Azure VM에서 SQL Server를 백업할 때 지원 설정 및 제한 사항에 대한 요약을 제공합니다.
ms.topic: conceptual
ms.date: 03/05/2020
ms.openlocfilehash: 79a7e30ab9240c489a66b547ff85bea7887131b1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79410000"
---
# <a name="support-matrix-for-sql-server-backup-in-azure-vms"></a>Azure VM에서 SQL 서버 백업에 대한 지원 매트릭스

Azure 백업을 사용하여 Microsoft Azure 클라우드 플랫폼에서 호스팅되는 Azure VM에서 SQL Server 데이터베이스를 백업할 수 있습니다. 이 문서에서는 Azure VM에서 SQL Server 백업의 시나리오 및 배포에 대한 일반적인 지원 설정 및 제한 사항에 대해 간략히 요약합니다.

## <a name="scenario-support"></a>시나리오 지원

**지원** | **세부 정보**
--- | ---
**지원되는 배포** | SQL Marketplace Azure VM 및 비 Marketplace VM(SQL Server가 수동 설치됨)이 지원됩니다.
**지원되는 지역** | 오스트레일리아 남동부(ASE), 오스트레일리아 동부(AE), 오스트레일리아 중부(AC), 오스트레일리아 중부 2(AC) <br> 브라질 남부(BRS)<br> 캐나다 중부(CNC), 캐나다 동부(CE)<br> 동남 아시아(SEA), 동아시아(EA) <br> 미국 동부(EUS), 미국 동부 2(EUS2), 미국 중서부(WCUS), 미국 서부(WUS), 미국 서부 2(WUS 2), 미국 중북부(NCUS), 미국 중부(CUS), 미국 중남부(SCUS) <br> 인도 중부(INC), 인도 남부(INS), 인도 서부 <br> 일본 동부(JPE), 일본 서부(JPW) <br> 한국 중부(KRC), 한국 남부(KRS) <br> 북유럽(NE), 서유럽 <br> 영국 남부(UKS), 영국 서부(UKW) <br> US Gov 애리조나, US Gov 버지니아, US Gov 텍사스, US DoD 중부, US DoD 동부 <br> 독일 북부, 독일 중서부 <br> 스위스 북부, 스위스 서부 <br> 프랑스 중부 <br> 중국 동부, 중국 동부 2, 중국 북부, 중국 북부 2
**지원되는 운영 체제** | Windows Server 2019, Windows Server 2016, Windows Server 2012, Windows Server 2008 R2 SP1 <br/><br/> Linux는 현재 지원되지 않습니다.
**지원되는 SQL Server 버전** | [제품 수명 주기 페이지 검색](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202017)에 설명된 SQL Server 2019, SQL Server 2017, [제품 수명 주기 페이지 검색](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202016%20service%20pack)에 설명된 SQL Server 2016 및 SP, SQL Server 2014, SQL Server 2012, SQL Server 2008 R2, SQL Server 2008 <br/><br/> Enterprise, Standard, Web, Developer, Express
**지원되는 .NET 버전** | VM에 설치된 .NET Framework 4.5.2 이상

## <a name="feature-consideration-and-limitations"></a>기능 고려 사항 및 제한 사항

* SQL Server 백업은 Azure Portal 또는 **PowerShell**에서 구성할 수 있습니다. CLI는 지원하지 않습니다.
* 솔루션은 Azure Resource Manager VM과 클래식 VM의 두 종류 [배포](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-model)에서 모두 지원됩니다.
* SQL Server를 실행하는 VM에서 Azure 공용 IP 주소에 액세스하려면 인터넷 연결이 필요합니다.
* SQL 서버 **장애 조치 클러스터 인스턴스(FCI)는** 지원되지 않습니다.
* 미러 데이터베이스와 데이터베이스 스냅샷에 대한 백업 및 복원 작업은 지원되지 않습니다.
* 백업 솔루션을 2개 이상 사용하여 독립 실행형 SQL Server 인스턴스 또는 SQL Always On 가용성 그룹을 백업하면 오류가 발생할 수 있으므로 그렇게 하지 말아야 합니다.
* 같은 솔루션 또는 다른 솔루션을 사용하여 한 가용성 그룹의 두 노드를 개별적으로 백업해도 오류가 발생할 수 있습니다.
* Azure Backup은 **읽기 전용** 데이터베이스에 전체 백업 및 복사 전용 전체 백업만 지원합니다.
* 많은 수의 파일이 있는 데이터베이스는 보호할 수 없습니다. 지원되는 최대 파일 수는 **1000**개입니다.  
* 볼트에서 ~ **2000개의** SQL Server 데이터베이스를 백업할 수 있습니다. 데이터베이스 수가 이보다 더 많은 경우 자격 증명 모음을 여러 개 만들면 됩니다.
* 한 번에 데이터베이스 **50**개까지 백업을 구성할 수 있습니다. 이 제한은 백업 부하 최적화에 도움이 됩니다.
* 우리는 최대 2 **TB크기의** 데이터베이스를 지원합니다. 성능 문제가 발생할 수 있습니다.
* 서버당 보호할 수 있는 데이터베이스 수를 파악하려면 대역폭, VM 크기, 백업 빈도, 데이터베이스 크기 등과 같은 요소를 고려해야 합니다. 리소스 플래너를 [다운로드하여](https://download.microsoft.com/download/A/B/5/AB5D86F0-DCB7-4DC3-9872-6155C96DE500/SQL%20Server%20in%20Azure%20VM%20Backup%20Scale%20Calculator.xlsx) VM 리소스 및 백업 정책에 따라 서버당 가질 수 있는 대략적인 데이터베이스 수를 계산합니다.
* 가용성 그룹이 구성되면 몇 가지 요인에 따라 다른 노드에서 백업이 수행됩니다. 아래는 가용성 그룹에 대한 백업 동작을 요약한 것입니다.

### <a name="back-up-behavior-with-always-on-availability-groups"></a>항상 가용성 그룹으로 비헤이비어 백업

백업은 AG(가용성 그룹)의 한 노드에서만 구성하는 것이 좋습니다. 항상 기본 노드와 동일한 리전에서 백업을 구성합니다. 즉, 백업을 구성하는 지역에 기본 노드가 항상 있어야 합니다. AG의 모든 노드가 백업이 구성된 동일한 지역에 있는 경우 걱정할 문제가 없습니다.

#### <a name="for-cross-region-ag"></a>지역 간 AG

* 백업 기본 설정에 관계없이 백업은 백업이 구성된 동일한 지역에 있는 노드에서만 실행됩니다. 이는 지역 간 백업이 지원되지 않기 때문입니다. 노드가 두 개만 있고 보조 노드가 다른 지역에 있는 경우 백업 기본 설정이 '보조 노드'인 경우가 아니면 기본 노드에서 백업이 계속 실행됩니다.
* 노드가 백업이 구성된 영역과 다른 지역으로 장애 조치되면 실패한 오버 지역의 노드에서 백업이 실패합니다.

백업 기본 설정 및 백업 유형(전체/차등/로그/복사 전용 전체)에 따라 특정 노드(주/보조)에서 백업이 수행됩니다.

#### <a name="backup-preference-primary"></a>백업 기본 설정: 기본

**백업 유형** | **노드**
--- | ---
전체 | 주
차등 | 주
로그 |  주
복사 전용 전체 |  주

#### <a name="backup-preference-secondary-only"></a>백업 기본 설정: 보조 전용

**백업 유형** | **노드**
--- | ---
전체 | 주
차등 | 주
로그 |  보조
복사 전용 전체 |  보조

#### <a name="backup-preference-secondary"></a>백업 기본 설정: 보조

**백업 유형** | **노드**
--- | ---
전체 | 주
차등 | 주
로그 |  보조
복사 전용 전체 |  보조

#### <a name="no-backup-preference"></a>백업 기본 설정 없음

**백업 유형** | **노드**
--- | ---
전체 | 주
차등 | 주
로그 |  보조
복사 전용 전체 |  보조

## <a name="next-steps"></a>다음 단계

Azure VM에서 실행 중인 [SQL Server 데이터베이스를 백업하는](backup-azure-sql-database.md) 방법을 알아봅니다.
