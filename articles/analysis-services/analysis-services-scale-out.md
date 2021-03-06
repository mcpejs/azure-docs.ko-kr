---
title: Azure Analysis Services 규모 확장 | Microsoft Docs
description: 확장을 통해 Azure 분석 서비스 서버를 복제합니다. 그런 다음 클라이언트 쿼리를 확장 쿼리 풀의 여러 쿼리 복제본 간에 배포할 수 있습니다.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 03/02/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 3ea304d038618fc428f20e7ad72b398f593d09a8
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78247984"
---
# <a name="azure-analysis-services-scale-out"></a>Azure Analysis Services 규모 확장

확장을 사용하면 쿼리 *풀의*여러 *쿼리 복제본* 간에 클라이언트 쿼리를 배포할 수 있어 높은 쿼리 워크로드 동안 응답 시간이 줄어듭니다. 또한 클라이언트 쿼리가 작업 처리에 의해 부정적인 영향을 받지 않도록 쿼리 풀에서 처리가 구분될 수도 있습니다. Azure Portal 또는 Analysis Services REST API를 사용하여 규모 확장을 구성할 수 있습니다.

규모 확장은 표준 가격 책정 계층의 서버에서 사용할 수 있습니다. 각 쿼리 복제본은 서버와 동일한 요금으로 청구됩니다. 모든 쿼리 복제본은 서버와 동일한 지역에 만들어집니다. 사용자가 구성할 수 있는 쿼리 복제본 수는 서버가 있는 지역에 따라 제한됩니다. 자세히 알아보려면 [지역별 가용성](analysis-services-overview.md#availability-by-region)을 참조하세요. 규모 확장은 서버의 사용 가능한 메모리 양을 증가시키지 않습니다. 메모리를 늘리려면 계획을 업그레이드해야 합니다. 

## <a name="why-scale-out"></a>왜 확장해야 합니까?

일반적인 서버 배포에서는 한 서버가 처리 서버 및 쿼리 서버 역할을 모두 수행합니다. 서버 모델에 대한 클라이언트 쿼리 수가 서버 계획에 대한 QPU(쿼리 처리 단위)를 초과하거나 높은 쿼리 워크로드와 동시에 모델 처리가 발생하면 성능이 저하될 수 있습니다. 

확장을 사용하면 최대 7개의 추가 쿼리 복제 *리소스(주* 서버를 포함하여 총 8개)로 쿼리 풀을 만들 수 있습니다. 중요한 시간에 QPU 요구를 충족하도록 쿼리 풀의 복제본 수를 확장할 수 있으며 언제든지 처리 서버를 쿼리 풀과 분리할 수 있습니다. 

쿼리 풀에 있는 쿼리 복제본의 수와 관계없이 처리 워크로드는 쿼리 복제본 간에 분산되지 않습니다. 기본 서버는 처리 서버역할을 합니다. 쿼리 복제본은 기본 서버와 쿼리 풀의 각 복제본 간에 동기화된 모델 데이터베이스에 대한 쿼리만 제공합니다. 

확장할 때 새 쿼리 복제본을 쿼리 풀에 증분적으로 추가하는 데 최대 5분이 걸릴 수 있습니다. 모든 새 쿼리 복제본이 실행중이면 쿼리 풀의 리소스 간에 새 클라이언트 연결이 로드 균형 조정됩니다. 기존 클라이언트 연결은 현재 연결되어 있는 리소스에서 변경되지 않습니다. 규모 감축 시 조정할 때 쿼리 풀에서 제거되는 쿼리 풀 리소스에 대한 기존 클라이언트 연결이 종료됩니다. 클라이언트는 나머지 쿼리 풀 리소스에 다시 연결할 수 있습니다.

## <a name="how-it-works"></a>작동 방법

처음으로 확장 을 구성할 때 기본 서버의 모델 데이터베이스가 새 쿼리 풀의 새 복제본과 *자동으로* 동기화됩니다. 자동 동기화는 한 번만 발생합니다. 자동 동기화 중에 기본 서버의 데이터 파일(Blob 저장소의 미사용 시 암호화)이 두 번째 위치로 복사되고 Blob 저장소의 미사용 데이터도 암호화됩니다. 그런 다음 쿼리 풀의 복제본은 두 번째 파일 집합의 데이터로 *수화됩니다.* 

서버를 처음 확장할 때만 자동 동기화가 수행되지만 수동 동기화를 수행할 수도 있습니다. 동기화는 쿼리 풀의 복제본에 대한 데이터를 기본 서버의 데이터와 일치시도록 합니다. 주 서버에서 모델을 새로 고칠 때 처리 작업이 완료된 *후* 동기화를 수행해야 합니다. 이 동기화는 Blob 저장소에 있는 기본 서버의 파일에서 두 번째 파일 집합으로 업데이트된 데이터를 복사합니다. 그런 다음 쿼리 풀의 복제본은 Blob 저장소의 두 번째 파일 집합에서 업데이트된 데이터로 수화됩니다. 

후속 확장 작업을 수행할 때(예: 쿼리 풀의 복제본 수를 2개에서 5개로 늘리는 경우 Blob 저장소의 두 번째 파일 집합의 데이터로 새 복제본이 수화됩니다). 동기화가 없습니다. 그런 다음 확장 한 후 동기화를 수행 하면 쿼리 풀의 새 복제본두 번 수화 될 것 이다-중복 된 수화. 후속 확장 작업을 수행할 때는 다음을 염두에 두어야 합니다.

* *추가된 복제본의* 중복 수화는 피하기 위해 확장 작업 전에 동기화를 수행합니다. 동시에 실행되는 동시 동기화 및 확장 작업이 허용되지 않습니다.

* 처리 *및* 확장 작업을 모두 자동화할 때는 먼저 기본 서버에서 데이터를 처리한 다음 동기화를 수행한 다음 확장 작업을 수행하는 것이 중요합니다. 이 시퀀스는 QPU 및 메모리 리소스에 미치는 영향을 최소화합니다.

* 쿼리 풀에 복제본이 없는 경우에도 동기화가 허용됩니다. 주 서버의 처리 작업의 새 데이터로 0에서 하나 이상의 복제본으로 확장하는 경우 먼저 쿼리 풀에 복제본이 없는 동기화를 수행한 다음 확장합니다. 확장하기 전에 동기화하면 새로 추가된 복제본의 중복 수화방지가 가능합니다.

* 기본 서버에서 모델 데이터베이스를 삭제할 때 쿼리 풀의 복제본에서 자동으로 삭제되지 않습니다. 동기화 작업을 수행 해야 [합니다 동기화-AzAnalysisServices인스턴스](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance) PowerShell 명령을 사용 하 여 복제본의 공유 Blob 저장소 위치에서 해당 데이터베이스에 대 한 파일/s를 제거 하 고 쿼리 풀에서 복제본에 모델 데이터베이스를 삭제 합니다. 모델 데이터베이스가 쿼리 풀의 복제본에 있지만 기본 서버에 없는지 확인하려면 **처리 서버가 쿼리 풀 설정에서** 분리된 경우 **예로**확인합니다. 그런 다음 SSMS를 사용하여 `:rw` 한정자를 사용하여 기본 서버에 연결하여 데이터베이스가 있는지 확인합니다. 그런 다음 `:rw` 한정자 없이 연결하여 쿼리 풀의 복제본에 연결하여 동일한 데이터베이스도 존재하는지 확인합니다. 데이터베이스가 쿼리 풀의 복제본에 있지만 기본 서버에는 없는 경우 동기화 작업을 실행합니다.   

* 주 서버에서 데이터베이스 이름을 바꿀 때 데이터베이스가 모든 복제본에 제대로 동기화되도록 하는 데 필요한 추가 단계가 있습니다. 이름을 변경한 후 이전 데이터베이스 이름으로 `-Database` 매개 변수를 지정하는 [Sync-AzAnalysisServicesInstance](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance) 명령을 사용하여 동기화를 수행합니다. 이 동기화는 모든 복제본에서 이전 이름의 데이터베이스및 파일을 제거합니다. 그런 다음 새 데이터베이스 이름으로 `-Database` 매개 변수를 지정하는 다른 동기화를 수행합니다. 두 번째 동기화는 새로 명명된 데이터베이스를 두 번째 파일 집합에 복사하고 복제본에 수화합니다. 포털에서 모델 동기화 명령을 사용하여 이러한 동기화를 수행할 수 없습니다.

### <a name="synchronization-mode"></a>동기화 모드

기본적으로 쿼리 복제본은 증분이 아닌 전체로 재수화됩니다. 재수화는 단계적으로 발생합니다. 한 번에 두 개의 복제본이 분리되고 첨부되어 지정된 시간에 쿼리를 위해 하나 이상의 복제본이 온라인 상태로 유지되도록 합니다. 경우에 따라 클라이언트는 이 프로세스가 진행되는 동안 온라인 복제본 중 하나에 다시 연결해야 할 수 있습니다. (미리 보기) **ReplicaSyncMode** 설정을 사용 하 여 이제 쿼리 복제본 동기화 가 병렬로 발생 지정할 수 있습니다. 병렬 동기화는 다음과 같은 이점을 제공합니다. 

- 동기화 시간이 크게 줄어듭니다. 
- 복제본 의 데이터는 동기화 프로세스 중에 일관되게 될 가능성이 높습니다. 
- 동기화 프로세스 전체의 모든 복제본에 데이터베이스가 온라인 상태로 유지되므로 클라이언트는 다시 연결할 필요가 없습니다. 
- 인메모리 캐시는 모델을 완전히 재수화하는 것보다 빠를 수 있는 변경된 데이터만으로 점진적으로 업데이트됩니다. 

#### <a name="setting-replicasyncmode"></a>복제본 싱크모드 설정

SSMS를 사용하여 고급 속성에서 복제SyncMode를 설정합니다. 사용 가능한 값은 

- `1`(기본값): 단계별로 전체 복제데이터베이스 재수화(증분). 
- `2`: 최적화된 동기화. 

![리리카싱크모드 설정](media/analysis-services-scale-out/aas-scale-out-sync-mode.png)

**ReplicaSyncMode=2를**설정할 때 업데이트해야 하는 캐시의 양에 따라 쿼리 복제본에서 추가 메모리를 사용할 수 있습니다. 데이터베이스를 온라인 상태로 유지하고 쿼리에 사용할 수 있도록 하려면 변경된 데이터의 양에 따라 이전 세그먼트와 새 세그먼트가 모두 동시에 메모리에 유지되므로 복제본의 *메모리를 최대 두 배로 늘려야* 할 수 있습니다. 복제본 노드는 기본 노드와 동일한 메모리 할당을 가지며 일반적으로 새로 고침 작업을 위해 기본 노드에 추가 메모리가 있으므로 복제본의 메모리가 부족할 가능성은 거의 없습니다. 또한 일반적인 시나리오는 데이터베이스가 기본 노드에서 점진적으로 업데이트되므로 메모리를 두 배로 늘리는 것이 일반적이어야 합니다. Sync 작업에 메모리 부족 오류가 발생하면 기본 기술(한 번에 두 개를 연결/분리)을 사용하여 다시 시도합니다. 

### <a name="separate-processing-from-query-pool"></a>쿼리 풀에서 처리 구분

처리 및 쿼리 작업 모두에서 최대 성능을 내기 위해서는 쿼리 풀에서 처리 서버를 분리하도록 선택할 수 있습니다. 분리되면 쿼리 풀의 쿼리 복제본에만 새 클라이언트 연결이 할당됩니다. 처리 작업에만 약간의 시간이 걸리는 경우 처리 및 동기화 작업을 수행하는 데 걸리는 약간의 시간 동안 쿼리 풀에서 처리 서버를 분리한 다음, 쿼리 풀에 다시 포함시키도록 선택할 수 있습니다. 처리 서버를 쿼리 풀에서 분리하거나 쿼리 풀에 다시 추가하는 경우 작업이 완료되는 데 최대 5분이 걸릴 수 있습니다.

## <a name="monitor-qpu-usage"></a>QPU 사용량 모니터링

서버에 대한 확장이 필요한지 확인하려면 메트릭을 사용하여 Azure 포털에서 [서버를 모니터링합니다.](analysis-services-monitor.md) QPU가 정기적으로 최대로 출력되는 지점까지 증가하는 경우 모델에 대한 쿼리 수가 계획의 QPU 한도를 초과하고 있음을 의미합니다. 쿼리 풀 작업 큐 길이 메트릭은 쿼리 스레드 풀 큐의 쿼리 수가 사용 가능한 QPU를 초과하면 증가합니다. 

또 다른 좋은 메트릭은 ServerResourceType의 평균 QPU입니다. 이 메트릭은 기본 서버의 평균 QPU와 쿼리 풀을 비교합니다. 

![메트릭 쿼리 확장](media/analysis-services-scale-out/aas-scale-out-monitor.png)

**서버리소스 유형별로 QPU를 구성하려면**

1. 메트릭 광고차트에서 메트릭 **추가를**클릭합니다. 
2. **RESOURCE에서**서버를 선택한 다음 **메트릭 네임스페이스에서** **분석 서비스 표준 메트릭을**선택한 다음 **메트릭에서** **QPU를**선택한 다음 **집계에서** **평균을**선택합니다. 
3. **분할 적용을 클릭합니다.** 
4. **값에서** **서버 리소스 유형을**선택합니다.  

### <a name="detailed-diagnostic-logging"></a>자세한 진단 로깅

확장된 서버 리소스에 대한 자세한 진단을 위해 Azure 모니터 로그를 사용합니다. 로그를 사용하면 Log Analytics 쿼리를 사용하여 서버 및 복제본별로 QPU 및 메모리를 나누기 할 수 있습니다. 자세한 내용은 [분석 서비스 진단 로깅의](analysis-services-logging.md#example-queries)예제 쿼리를 참조하십시오.


## <a name="configure-scale-out"></a>규모 확장 구성

### <a name="in-azure-portal"></a>Azure Portal에서

1. 포털에서 배율 조정 을 **클릭합니다.** 슬라이더를 사용하여 쿼리 복제 서버 수를 선택합니다. 선택한 복제본 수는 기존 서버에 추가됩니다.  

2. 쿼리 서버에서 처리 서버를 제외하려면 **쿼리 풀에서 처리 서버 구분**에서 [예]를 선택합니다. 기본 연결 문자열(있음)을 `:rw`사용하는 클라이언트 [연결은](#connections) 쿼리 풀의 복제본으로 리디렉션됩니다. 

   ![규모 확장 슬라이더](media/analysis-services-scale-out/aas-scale-out-slider.png)

3. 새 쿼리 복제본 서버를 프로비전하려면 **저장**을 클릭합니다. 

서버에 대해 처음으로 확장 을 구성할 때 기본 서버의 모델이 쿼리 풀의 복제본과 자동으로 동기화됩니다. 자동 동기화는 하나 이상의 복제본에 대해 확장 을 처음 구성할 때 한 번만 발생합니다. 동일한 서버의 복제본 수를 변경하면 *다른 자동 동기화가 트리거되지 않습니다.* 서버를 복제본 0개로 설정한 다음 여러 복제본으로 다시 확장해도 자동 동기화가 다시 발생하지 않습니다. 

## <a name="synchronize"></a>동기화 

동기화 작업은 수동으로 또는 REST API를 사용하여 수행해야 합니다.

### <a name="in-azure-portal"></a>Azure Portal에서

**개요** > 모델 > **모델 동기화**에서.

![규모 확장 슬라이더](media/analysis-services-scale-out/aas-scale-out-sync.png)

### <a name="rest-api"></a>REST API

**동기화** 작업을 사용합니다.

#### <a name="synchronize-a-model"></a>모델 동기화   

`POST https://<region>.asazure.windows.net/servers/<servername>:rw/models/<modelname>/sync`

#### <a name="get-sync-status"></a>동기화 상태 가져오기  

`GET https://<region>.asazure.windows.net/servers/<servername>/models/<modelname>/sync`

상태 코드 반환:


|코드  |설명  |
|---------|---------|
|-1     |  올바르지 않음       |
|0     | Replicating        |
|1     |  재수화       |
|2     |   Completed       |
|3     |   실패      |
|4     |    마무리     |
|||


### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

PowerShell을 사용하기 전에 [최신 Azure PowerShell 모듈을 설치하거나 업데이트합니다.](/powershell/azure/install-az-ps) 

동기화를 실행하려면 [동기화-AzAnalysisServices인스턴스](https://docs.microsoft.com/powershell/module/az.analysisservices/sync-AzAnalysisServicesinstance)를 사용합니다.

쿼리 복제본 수를 설정하려면 [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver)를 사용합니다. 선택적 `-ReadonlyReplicaCount` 매개 변수를 지정합니다.

처리 서버를 쿼리 풀과 분리하려면 [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver)를 사용합니다. 사용할 선택적 `-DefaultConnectionMode` 매개 `Readonly`변수를 지정합니다.

자세한 내용은 [Az.AnalysisServices 모듈을 사용하여 서비스 주체 사용(보안 주체 사용)을](analysis-services-service-principal.md#azmodule)참조하십시오.

## <a name="connections"></a>Connections

서버의 [개요] 페이지에는 두 개의 서버 이름이 있습니다. 서버에 대한 규모 확장을 아직 구성하지 않은 경우 두 서버 이름은 동일하게 작동합니다. 서버에 대한 규모 확장을 구성한 후에는 연결 형식에 따라 적절한 서버 이름을 지정해야 합니다. 

Power BI 데스크톱, Excel 및 사용자 지정 앱과 같은 최종 사용자 클라이언트 연결의 경우 **서버 이름을**사용합니다. 

PowerShell, Azure Function 앱 및 AMO의 SSMS, Visual Studio 및 연결 문자열의 경우 **관리 서버 이름을**사용합니다. 관리 서버 이름에는 특별한 `:rw`(읽기/쓰기) 한정자가 포함됩니다. 모든 처리 작업은 (기본) 관리 서버에서 발생합니다.

![서버 이름](media/analysis-services-scale-out/aas-scale-out-name.png)

## <a name="scale-up-scale-down-vs-scale-out"></a>확장, 확장/ 축소 vs. 확장

복제본이 여러 개인 서버에서 가격 책정 계층을 변경할 수 있습니다. 동일한 가격 책정 계층이 모든 복제본에 적용됩니다. 규모 조정 작업은 먼저 모든 복제본을 한 번에 모두 축소한 다음 새 가격 책정 계층의 모든 복제본을 가져옵니다.

## <a name="troubleshoot"></a>문제 해결

**문제:** 사용자는 **연결 모드 'ReadOnly'에서 '\<서버 이름>' 서버 인스턴스를 찾을 수 없습니다.** 라는 오류를 받게 됩니다.

**솔루션:** **쿼리 풀 옵션에서 처리 서버 분리를** 선택하면 기본 연결 문자열을 `:rw`사용하는 클라이언트 연결(없는 경우)이 쿼리 풀 복제본으로 리디렉션됩니다. 동기화가 완료되지 않았기 때문에 쿼리 풀의 복제본이 아직 온라인 상태가 아니면 리디렉션된 클라이언트 연결이 실패할 수 있습니다. 연결 실패를 방지하려면 동기화를 수행할 때 쿼리 풀에 두 개 이상의 서버가 있어야 합니다. 다른 서버가 온라인 상태로 유지되는 동안 각 서버는 개별적으로 동기화됩니다. 처리 중에 쿼리 풀에 처리 서버가 없도록 선택한 경우 처리를 위한 풀에서 처리 서버를 제거한 다음, 처리가 완료된 후 동기화되기 전, 다시 풀에 추가하도록 선택할 수 있습니다. 메모리 및 QPU 메트릭을 사용하여 동기화 상태를 모니터링할 수 있습니다.



## <a name="related-information"></a>관련 정보

[서버 메트릭 모니터링](analysis-services-monitor.md)   
[Azure Analysis Services 관리](analysis-services-manage.md) 
