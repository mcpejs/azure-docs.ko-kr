---
title: Anomaly Detector API를 사용하는 경우 모범 사례
titleSuffix: Azure Cognitive Services
description: 변칙 탐지기 API를 통해 이상 징후를 검색할 때 모범 사례에 대해 알아봅니다.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 03/26/2019
ms.author: aahi
ms.openlocfilehash: 9407f2fc9375765efb6eb9688b3ebfeef24ba90a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "67721631"
---
# <a name="best-practices-for-using-the-anomaly-detector-api"></a>변칙 탐지기 API 사용에 대한 모범 사례

변칙 검출기 API는 상태 비수기 변칙 검색 서비스입니다. 결과의 정확성과 성능은 다음과 같은 영향을 받을 수 있습니다.

* 타임시리즈 데이터가 준비되는 방식.
* 사용된 변칙 검출기 API 매개 변수입니다.
* API 요청의 데이터 소수점입니다. 

이 문서를 사용하여 API를 사용하여 데이터에 대한 최상의 결과를 얻는 모범 사례에 대해 알아봅니다. 

## <a name="when-to-use-batch-entire-or-latest-last-point-anomaly-detection"></a>배치(전체) 또는 최신(마지막) 점 변칙 검색을 사용하는 경우

변칙 검출기 API의 배치 감지 엔드포인트를 사용하면 전체 시간 계열 데이터를 통해 이상 징후를 감지할 수 있습니다. 이 검색 모드에서는 단일 통계 모델이 만들어지고 데이터 집합의 각 점에 적용됩니다. 열렬에 아래 특성이 있는 경우 일괄 처리 검색을 사용하여 하나의 API 호출에서 데이터를 미리 볼 수 있습니다.

* 계절별 타임시리즈로, 가끔 이상현상이 있습니다.
* 플랫 트렌드 타임 시리즈, 가끔 스파이크 / 딥. 

실시간 데이터 모니터링을 위해 일괄 처리 변칙 검색을 사용하거나 위의 특성이 없는 열렬 데이터에 는 사용하지 않는 것이 좋습니다. 

* 일괄 처리 감지는 하나의 모델만 생성하고 적용하며, 각 점에 대한 검색은 전체 계열의 컨텍스트에서 수행됩니다. 시간계 데이터가 계절성 없이 위아래로 추세가 나타나면 일부 변경 지점(데이터의 급락 및 스파이크)이 모델에서 누락될 수 있습니다. 마찬가지로 데이터 집합의 후반부보다 덜 중요한 일부 변경 지점은 모델에 통합될 만큼 중요하지 않을 수 있습니다.

* 일괄 처리 감지는 분석되는 포인트 수로 인해 실시간 데이터 모니터링을 수행할 때 최신 지점의 이상 상태를 감지하는 것보다 느립니다.

실시간 데이터 모니터링의 경우 최신 데이터 포인트의 이상 상태를 검색하는 것이 좋습니다. 최신 포인트 감지를 지속적으로 적용하면 스트리밍 데이터 모니터링을 보다 효율적이고 정확하게 수행할 수 있습니다.

아래 예제에서는 이러한 검색 모드가 성능에 미칠 수 있는 영향을 설명합니다. 첫 번째 그림은 이전에 본 28개의 데이터 포인트를 따라 변칙 상태 최신 지점을 지속적으로 감지한 결과를 보여줍니다. 빨간색 점은 이상입니다.

![최신 점을 사용하여 변칙 검색을 보여주는 이미지](../media/last.png)

다음은 일괄 처리 변칙 검색을 사용하는 동일한 데이터 집합입니다. 작업에 대해 빌드된 모델은 사각형으로 표시된 몇 가지 예외를 무시했습니다.

![배치 메서드를 사용하여 변칙 검색을 보여 주면](../media/entire.png)

## <a name="data-preparation"></a>데이터 준비

변칙 검출기 API는 JSON 요청 개체에 서식이 지정된 시계 데이터를 허용합니다. 타임계는 시간에 따라 순차적으로 기록된 모든 숫자 데이터가 될 수 있습니다. API의 성능을 향상시키기 위해 이상 징후 탐지기 API 끝점으로 열렬 데이터의 창을 보낼 수 있습니다. 보낼 수 있는 최소 데이터 포인트 수는 12개이며 최대 포인트는 8640점입니다. [세분성은](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.anomalydetector.models.granularity?view=azure-dotnet-preview) 데이터가 샘플링되는 비율로 정의됩니다. 

변칙 검출기 API로 전송된 데이터 포인트에는 유효한 UTC(조정된 유니버설 타임) 타임스탬프와 숫자 값이 있어야 합니다. 

```json
{
    "granularity": "daily",
    "series": [
      {
        "timestamp": "2018-03-01T00:00:00Z",
        "value": 32858923
      },
      {
        "timestamp": "2018-03-02T00:00:00Z",
        "value": 29615278
      },
    ]
}
```

데이터가 비표준 시간 간격으로 샘플링된 경우 요청에 특성을 `customInterval` 추가하여 데이터를 지정할 수 있습니다. 예를 들어 5분마다 시리즈를 샘플링하는 경우 JSON 요청에 다음을 추가할 수 있습니다.

```json
{
    "granularity" : "minutely", 
    "customInterval" : 5
}
```

### <a name="missing-data-points"></a>누락된 데이터 점

누락된 데이터 요소가 균등하게 분산된 열열 데이터 집합, 특히 미세한 세분성(작은 샘플링 간격)이 있는 데이터 집합에서 일반적입니다. 예를 들어 몇 분마다 샘플링된 데이터)를 예로 들 수 있습니다. 데이터에서 예상 포인트 수의 10% 미만을 누락하면 검색 결과에 부정적인 영향을 미치지 않습니다. 이전 기간의 데이터 포인트 대체, 선형 보간 또는 이동 평균과 같은 특성에 따라 데이터의 간격을 채우는 것이 좋습니다.

### <a name="aggregate-distributed-data"></a>분산 데이터 집계

변칙 검출기 API는 균등하게 분산된 시간열에서 가장 잘 작동합니다. 데이터가 임의로 분산된 경우 분당, 시간별 또는 일일(예: 시간 단위)으로 데이터를 집계해야 합니다.

## <a name="anomaly-detection-on-data-with-seasonal-patterns"></a>계절적 패턴이 있는 데이터에 대한 이상 징후 감지

열계 데이터에 계절 적 패턴(정기적으로 발생하는 패턴)이 있는 경우 정확도 및 API 응답 시간을 향상시킬 수 있습니다. 

JSON `period` 요청을 생성할 때 를 지정하면 변칙 검색 대기 시간이 최대 50%까지 줄어들 수 있습니다. 는 `period` 패턴을 반복하는 데 걸리는 시간계의 데이터 포인트 수를 대략 지정하는 정수입니다. 예를 들어 하루에 하나의 데이터 포인트가 있는 `period` 시계열에는 as가 `7`있고 시간당 1포인트(동일한 주간 패턴)가 `7*24`있는 시계열에는 의 가가 `period` 있습니다. 데이터 패턴을 잘 모르는 경우 이 매개 변수를 지정할 필요가 없습니다.

최상의 결과를 얻으려면 `period`4's 의 데이터 포인트와 추가 데이터 포인트를 제공합니다. 예를 들어, 위에서 설명한 대로 주간 패턴이 있는 시간별 데이터는 요청`7 * 24 * 4 + 1`본문()에 673개의 데이터 포인트를 제공해야 합니다.

### <a name="sampling-data-for-real-time-monitoring"></a>실시간 모니터링을 위한 데이터 샘플링

스트리밍 데이터가 짧은 간격(예: 초 또는 분)으로 샘플링되는 경우 권장 데이터 포인트 수를 전송하는 것이 변칙 탐지기 API의 허용된 최대 수(8640데이터 포인트)를 초과할 수 있습니다. 데이터에 안정적인 계절 패턴이 표시되는 경우 시간(시간)과 같이 더 큰 시간 간격으로 타임계 데이터의 샘플을 보내는 것이 좋습니다. 이러한 방식으로 데이터를 샘플링하면 API 응답 시간이 눈에 띄게 향상될 수 있습니다. 

## <a name="next-steps"></a>다음 단계

* [Anomaly Detector API란?](../overview.md)
* [빠른 시작: 변칙 검출기 REST API를 사용하여 열렬 데이터의 이상 징후 감지](../quickstarts/detect-data-anomalies-csharp.md)
