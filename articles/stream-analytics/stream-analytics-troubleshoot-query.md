---
title: Azure Stream Analytics 쿼리 문제 해결
description: 이 문서에서는 Azure Stream Analytics 작업에서 쿼리 문제를 해결하는 기술에 대해 설명합니다.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/31/2020
ms.custom: seodec18
ms.openlocfilehash: f049dc6d1261a8201cf79d1779e522b30d13c4b0
ms.sourcegitcommit: 27bbda320225c2c2a43ac370b604432679a6a7c0
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2020
ms.locfileid: "80409449"
---
# <a name="troubleshoot-azure-stream-analytics-queries"></a>Azure Stream Analytics 쿼리 문제 해결

이 문서에서는 Stream Analytics 쿼리 개발과 관련한 일반적인 문제 및 그 해결 방법을 설명합니다.

이 문서에서는 Azure Stream Analytics 쿼리 개발과 관련된 일반적인 문제, 쿼리 문제 해결 방법 및 문제를 해결하는 방법에 대해 설명합니다. 많은 문제 해결 단계를 수행하려면 Stream Analytics 작업에 진단 로그를 사용하도록 설정해야 합니다. 진단 로그를 사용하도록 설정하지 않은 경우 [진단 로그를 사용하여 Azure Stream 분석 문제를 해결합니다.](stream-analytics-job-diagnostic-logs.md)

## <a name="query-is-not-producing-expected-output"></a>쿼리가 예상 출력을 생성하지 않음

1.  로컬로 테스트하여 오류를 검사합니다.

    - Azure 포털에서 **쿼리** 탭에서 **테스트를**선택합니다. 다운로드한 샘플 데이터를 사용하여 [쿼리를 테스트](stream-analytics-test-query.md)합니다. 모든 오류를 검사하고 수정합니다.   
    - 또한 Visual Studio 또는 [Visual Studio 코드에](visual-studio-code-local-run-live-input.md)대한 Azure Stream Analytics 도구를 사용하여 [쿼리를 로컬로 테스트할](stream-analytics-live-data-local-testing.md) 수도 있습니다. 

2.  Visual Studio용 Azure 스트림 분석 도구에서 [작업 다이어그램을 사용하여 쿼리를 로컬로 단계별로 디버그합니다.](debug-locally-using-job-diagram.md) 작업 다이어그램은 여러 쿼리 단계를 통해 입력 소스(이벤트 허브, IoT Hub 등)에서 데이터가 흐르고 마지막으로 싱크를 출력하는 방법을 보여 주며, 마지막으로 싱크를 출력하는 방법을 보여 주며, 각 쿼리 단계는 WITH 문을 사용하여 스크립트에 정의된 임시 결과 집합에 매핑됩니다. 각 중간 결과 집합에서 데이터와 메트릭을 보고 문제의 원인을 찾을 수 있습니다.

    ![작업 다이어그램 미리 보기 결과](./media/debug-locally-using-job-diagram/preview-result.png)

3.  [**Timestamp By**](https://docs.microsoft.com/stream-analytics-query/timestamp-by-azure-stream-analytics)를 사용하는 경우 이벤트에 [작업 시작 시간](stream-analytics-out-of-order-and-late-events.md)보다 큰 타임스탬프가 있는지 확인합니다.

4.  다음과 같은 공통 문제를 제거합니다.
    - 쿼리의 [**WHERE**](https://docs.microsoft.com/stream-analytics-query/where-azure-stream-analytics) 절은 출력이 생성되지 않도록 방지하는 모든 이벤트를 필터링했습니다.
    - [**캐스트**](https://docs.microsoft.com/stream-analytics-query/cast-azure-stream-analytics) 기능이 실패하면 작업이 실패합니다. 형식 캐스팅 오류를 방지하려면 [**TRY_CAST**](https://docs.microsoft.com/stream-analytics-query/try-cast-azure-stream-analytics)를 대신 사용하세요.
    - 창 함수를 사용하는 경우 전체 창 기간을 기다려서 쿼리의 출력을 확인합니다.
    - 이벤트의 타임스탬프는 작업 시작 시간 앞에 있으며 이벤트가 삭제됩니다.
    - [**JOIN**](https://docs.microsoft.com/stream-analytics-query/join-azure-stream-analytics) 조건이 일치하지 않습니다. 일치하는 항목이 없는 경우 출력이 0이 됩니다.

5.  이벤트 순서 지정 정책이 예상대로 구성되었는지 확인합니다. **설정으로** 이동하여 [**이벤트 순서를 선택합니다.**](stream-analytics-out-of-order-and-late-events.md) 정책은 **테스트** 단추를 사용하여 쿼리를 테스트하는 경우 적용되지 *않습니다*. 이것이 브라우저에서 테스트할 때와 프로덕션의 작업을 실행할 때의 차이입니다. 

6. 감사 및 진단 로그를 사용하여 디버그합니다.
    - [감사 로그](../azure-resource-manager/resource-group-audit.md)를 사용하고 필터링하여 오류를 식별하고 디버그합니다.
    - [작업 진단 로그](stream-analytics-job-diagnostic-logs.md)를 사용하여 오류를 식별하고 디버그합니다.

## <a name="resource-utilization-is-high"></a>리소스 사용률이 높습니다.

Azure Stream Analytics의 병렬 처리를 활용해야 합니다. 입력 파티션을 구성하고, 분석 쿼리 정의를 조정하여 Stream Analytics 작업의 [쿼리 병렬 처리로 크기를 조정](stream-analytics-parallelization.md)할 수 있습니다.

## <a name="debug-queries-progressively"></a>점진적으로 쿼리 디버그

실시간 데이터를 처리하는 동안 쿼리 중간에 데이터가 어떤 상태인지를 알면 유용할 수 있습니다. Visual Studio의 작업 다이어그램을 사용하여 이를 볼 수 있습니다. Visual Studio가 없는 경우 중간 데이터를 출력하기 위한 추가 단계를 수행할 수 있습니다.

Azure Stream Analytics 작업의 입력 또는 단계를 여러 번 읽을 수 있기 때문에 추가 SELECT INTO 문을 작성할 수 있습니다. 이렇게 하면 프로그램을 디버그할 때 *watch 변수*가 동작하는 것처럼 중간 데이터가 스토리지로 출력되어 데이터의 정확성을 검사할 수 있습니다.

Azure Stream Analytics 작업의 다음 예제 쿼리에는 Azure Table Storage에 대해 하나의 스트림 입력, 두 개의 참조 데이터 입력 및 출력이 있습니다. 쿼리는 이벤트 허브의 데이터와 두 개의 참조 Blob을 조인하여 이름 및 범주 정보를 가져옵니다.

![Stream Analytics SELECT INTO 쿼리 예제](./media/stream-analytics-select-into/stream-analytics-select-into-query1.png)

작업이 실행되고 있지만 이벤트는 출력에 생성되지 않습니다. 여기에 표시된 **모니터링** 타일에서 입력이 데이터를 생성하는 것을 확인할 수는 있지만 **JOIN**의 어떤 단계에서 모든 이벤트가 삭제되었는지는 알 수 없습니다.

![Stream Analytics 모니터링 타일](./media/stream-analytics-select-into/stream-analytics-select-into-monitor.png)

이 경우 몇 가지 추가 SELECT INTO 문을 추가하여 중간 JOIN 결과와 입력에서 읽은 데이터를 “로깅”할 수 있습니다.

이 예제에서는 두 개의 새로운 “임시 출력”을 추가했습니다. 사용자가 원하는 어떠한 싱크도 될 수 있습니다. 여기서는 Azure Storage를 예로 사용합니다.

![Stream Analytics 쿼리에 SELECT INTO 문 추가](./media/stream-analytics-select-into/stream-analytics-select-into-outputs.png)

그런 다음 다음과 같이 쿼리를 다시 작성할 수 있습니다.

![다시 작성된 SELECT INTO Stream Analytics 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-query2.png)

이제 작업을 다시 시작하고 몇 분 동안 실행되도록 합니다. 그런 다음 Visual Studio Cloud Explorer를 통해 temp1 및 temp2를 쿼리하여 다음과 같은 테이블을 생성합니다.

**temp1 테이블**
![SELECT INTO temp1 테이블 스트림 분석 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-1.png)

**temp2 테이블**
![SELECT INTO temp2 테이블 스트림 분석 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-2.png)

보시다시피 temp1 및 temp2에 모두 데이터가 있고 이름 열이 temp2에서 올바르게 채워집니다. 그러나 여전히 출력에 데이터가 없기 때문에 문제가 있습니다.

![데이터 Stream Analytics 쿼리가 없는 SELECT INTO output1 테이블](./media/stream-analytics-select-into/stream-analytics-select-into-out-table-1.png)

데이터를 샘플링하면 두 번째 JOIN에서 문제가 있음을 거의 확신할 수 있습니다. Blob의 참조 데이터를 다운로드하고 살펴 볼 수 있습니다.

![SELECT INTO 참조 테이블 Stream Analytics 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-1.png)

보시다시피 이 참조 데이터에서 GUID의 형식은 temp2의 열 형식과는 다릅니다. 데이터가 output1에 예상대로 도착하지 않았기 때문입니다.

데이터 형식을 수정하고, 참조 Blob에 업로드하고 다시 시도할 수 있습니다.

![SELECT INTO 임시 테이블 Stream Analytics 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-2.png)

이번에는 출력 데이터가 예상대로 포맷되고 채워져 있습니다.

![SELECT INTO 최종 테이블 Stream Analytics 쿼리](./media/stream-analytics-select-into/stream-analytics-select-into-final-table.png)

## <a name="get-help"></a>도움말 보기

추가 지원은 [Azure 스트림 분석 포럼을](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)참조하십시오.

## <a name="next-steps"></a>다음 단계

* [Azure Stream Analytics 소개](stream-analytics-introduction.md)
* [Azure 스트림 분석 사용 시작](stream-analytics-real-time-fraud-detection.md)
* [Azure  Stream Analytics 작업 규모 지정](stream-analytics-scale-jobs.md)
* [Azure 스트림 분석 쿼리 언어 참조](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Azure Stream Analytics 관리 REST API 참조](https://msdn.microsoft.com/library/azure/dn835031.aspx)
