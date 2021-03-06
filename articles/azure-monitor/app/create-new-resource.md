---
title: 새 Azure Application Insights 리소스 만들기 | Microsoft Docs
description: 새 라이브 애플리케이션에 대한 Application Insights 모니터링을 수동으로 설정합니다.
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: c1b3a6920723ad59b714cce4bd69e1b95fe1995f
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80132414"
---
# <a name="create-an-application-insights-resource"></a>Application Insights 리소스 만들기

Azure Application Insights는 Microsoft Azure *리소스*에 애플리케이션에 대한 데이터를 표시합니다. 따라서 새 리소스 만들기는 [새 애플리케이션을 모니터링하도록 Application Insights를 설정][start]하는 과정에 포함됩니다. 새 리소스를 만든 후 계측 키를 얻고 이를 사용하여 응용 프로그램 인사이트 SDK를 구성할 수 있습니다. 계측 키는 원격 분석을 리소스에 연결합니다.

## <a name="sign-in-to-microsoft-azure"></a>Microsoft Azure에 로그인

Azure 구독이 없는 경우 시작하기 전에 [무료](https://azure.microsoft.com/free/) 계정을 만드세요.

## <a name="create-an-application-insights-resource"></a>Application Insights 리소스 만들기

[Azure 포털에](https://portal.azure.com)로그인하고 응용 프로그램 인사이트 리소스를 만듭니다.

![왼쪽 상단 모서리에 있는 '+' 기호를 클릭합니다. 개발자 도구 선택 후 애플리케이션 인사이트](./media/create-new-resource/new-app-insights.png)

   | 설정        |  값           | 설명  |
   | ------------- |:-------------|:-----|
   | **이름**      | 고유 값 | 모니터링 중인 앱을 식별하는 이름입니다. |
   | **리소스 그룹**     | myResourceGroup      | App Insights 데이터를 호스트할 새 리소스 또는 기존 리소스 그룹의 이름입니다. |
   | **위치** | 미국 동부 | 가까운 위치 또는 앱이 호스팅되는 위치 근처를 선택합니다. |

> [!NOTE]
> 다른 리소스 그룹에서 동일한 리소스 이름을 사용할 수 있지만 전역고유의 이름을 사용하는 것이 좋습니다. 이 기능은 필요한 구문을 단순화할 때 [교차 리소스 쿼리를 수행하려는](https://docs.microsoft.com/azure/azure-monitor/log-query/cross-workspace-query#identifying-an-application) 경우에 유용할 수 있습니다.

필요한 필드에 적절한 값을 입력한 다음 **검토 + 만들기를**선택합니다.

![필수 필드에 값을 입력한 다음 "검토 + 만들기"를 선택합니다.](./media/create-new-resource/review-create.png)

앱을 만들면 새 창이 열립니다. 이 창은 모니터링되는 응용 프로그램에 대한 성능 및 사용 현황 데이터를 볼 수 있는 창입니다. 

## <a name="copy-the-instrumentation-key"></a>계측 키 복사

계측 키는 원격 분석 데이터를 연결할 리소스를 식별합니다. 계측 키를 복사하여 응용 프로그램의 코드에 추가해야 합니다.

![계측 키를 클릭하고 복사합니다.](./media/create-new-resource/instrumentation-key.png)

## <a name="install-the-sdk-in-your-app"></a>응용 프로그램에 SDK를 설치합니다.

Application Insights SDK를 애플리케이션에 설치합니다. 이 단계는 애플리케이션의 형식에 따라 크게 달라 집니다.

계측 키를 사용하여 [애플리케이션에 설치한 SDK][start]를 구성합니다.

SDK에는 추가 코드를 작성하지 않고도 원격 분석을 보내는 표준 모듈이 포함되어 있습니다. 사용자 작업을 추적하거나 문제를 보다 세부적으로 진단하려면 [API를 사용][api]하여 사용자 고유의 원격 분석을 보내도록 합니다.

## <a name="creating-a-resource-automatically"></a>자동으로 리소스 만들기

### <a name="powershell"></a>PowerShell

새 Application Insights 리소스 만들기

```powershell
New-AzApplicationInsights [-ResourceGroupName] <String> [-Name] <String> [-Location] <String> [-Kind <String>]
 [-Tag <Hashtable>] [-DefaultProfile <IAzureContextContainer>] [-WhatIf] [-Confirm] [<CommonParameters>]
```

#### <a name="example"></a>예제

```powershell
New-AzApplicationInsights -Kind java -ResourceGroupName testgroup -Name test1027 -location eastus
```
#### <a name="results"></a>결과

```powershell
Id                 : /subscriptions/{subid}/resourceGroups/testgroup/providers/microsoft.insights/components/test1027
ResourceGroupName  : testgroup
Name               : test1027
Kind               : web
Location           : eastus
Type               : microsoft.insights/components
AppId              : 8323fb13-32aa-46af-b467-8355cf4f8f98
ApplicationType    : web
Tags               : {}
CreationDate       : 10/27/2017 4:56:40 PM
FlowType           :
HockeyAppId        :
HockeyAppToken     :
InstrumentationKey : 00000000-aaaa-bbbb-cccc-dddddddddddd
ProvisioningState  : Succeeded
RequestSource      : AzurePowerShell
SamplingPercentage :
TenantId           : {subid}
```

이 cmdlet에 대한 전체 PowerShell 설명서와 계측 기중키를 검색하는 방법에 대한 자세한 내용은 [Azure PowerShell 설명서를](https://docs.microsoft.com/powershell/module/az.applicationinsights/new-azapplicationinsights?view=azps-2.5.0)참조하십시오.

### <a name="azure-cli-preview"></a>Azure CLI(미리 보기)

미리 보기에 액세스하려면 응용 프로그램 인사이트 Azure CLI 명령을 먼저 실행해야 합니다.

```azurecli
 az extension add -n application-insights
```

`az extension add` 명령을 실행하지 않으면 다음과 같은 오류 메시지가 표시됩니다.`az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

이제 다음을 실행하여 응용 프로그램 인사이트 리소스를 만들 수 있습니다.

```azurecli
az monitor app-insights component create --app
                                         --location
                                         --resource-group
                                         [--application-type]
                                         [--kind]
                                         [--tags]
```

#### <a name="example"></a>예제

```azurecli
az monitor app-insights component create --app demoApp --location westus2 --kind web -g demoRg --application-type web
```

#### <a name="results"></a>결과

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g demoApp  --application-type web
{
  "appId": "87ba512c-e8c9-48d7-b6eb-118d4aee2697",
  "applicationId": "demoApp",
  "applicationType": "web",
  "creationDate": "2019-08-16T18:15:59.740014+00:00",
  "etag": "\"0300edb9-0000-0100-0000-5d56f2e00000\"",
  "flowType": "Bluefield",
  "hockeyAppId": null,
  "hockeyAppToken": null,
  "id": "/subscriptions/{subid}/resourceGroups/demoApp/providers/microsoft.insights/components/demoApp",
  "instrumentationKey": "00000000-aaaa-bbbb-cccc-dddddddddddd",
  "kind": "web",
  "location": "eastus",
  "name": "demoApp",
  "provisioningState": "Succeeded",
  "requestSource": "rest",
  "resourceGroup": "demoApp",
  "samplingPercentage": null,
  "tags": {},
  "tenantId": {tenantID},
  "type": "microsoft.insights/components"
}
```

이 명령에 대한 전체 Azure CLI 설명서및 계측 기중기 키를 검색하는 방법에 대한 자세한 내용은 [Azure CLI 설명서를](https://docs.microsoft.com/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-create)참조하십시오.

## <a name="next-steps"></a>다음 단계
* [진단 검색](../../azure-monitor/app/diagnostic-search.md)
* [메트릭 탐색](../../azure-monitor/app/metrics-explorer.md)
* [분석 쿼리 작성](../../azure-monitor/app/analytics.md)

<!--Link references-->

[api]: ../../azure-monitor/app/api-custom-events-metrics.md
[diagnostic]: ../../azure-monitor/app/diagnostic-search.md
[metrics]: ../../azure-monitor/app/metrics-explorer.md
[start]: ../../azure-monitor/app/app-insights-overview.md
