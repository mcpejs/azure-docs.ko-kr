---
title: 지속성 함수의 오류 처리 - Azure
description: Azure Functions의 지속성 함수 확장에서 오류를 처리하는 방법을 알아봅니다.
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: azfuncdf
ms.openlocfilehash: 447b3dcf5040835f5a853beff68bde794ece51f5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79277857"
---
# <a name="handling-errors-in-durable-functions-azure-functions"></a>지속성 함수의 오류 처리(Azure Functions)

지속형 함수 오케스트레이션은 코드에서 구현되며 프로그래밍 언어의 기본 제공 오류 처리 기능을 사용할 수 있습니다. 오케스트레이션에 오류 처리 및 보정을 추가하는 방법을 배워야 하는 새로운 개념은 없습니다. 그러나 알고 있어야 하는 몇 가지 동작이 있습니다.

## <a name="errors-in-activity-functions"></a>작업 함수의 오류

활동 함수에 throw된 모든 예외는 오케스트레이터 함수로 다시 마샬링되고 `FunctionFailedException`로 throw됩니다. 오케스트레이터 함수에서 요구 사항에 적합한 오류 처리 및 보정 코드를 작성할 수 있습니다.

예를 들어 한 계정에서 다른 계정으로 자금을 이체하는 다음 오케스트레이터 함수를 살펴보세요.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("TransferFunds")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var transferDetails = context.GetInput<TransferOperation>();

    await context.CallActivityAsync("DebitAccount",
        new
        {
            Account = transferDetails.SourceAccount,
            Amount = transferDetails.Amount
        });

    try
    {
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.DestinationAccount,
                Amount = transferDetails.Amount
            });
    }
    catch (Exception)
    {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.SourceAccount,
                Amount = transferDetails.Amount
            });
    }
}
```

> [!NOTE]
> 이전 C# 예제는 지속 기능 2.x에 대 한 입니다. 지속 기능 1.x의 경우 `DurableOrchestrationContext` `IDurableOrchestrationContext`을 대신 사용해야 합니다. 버전 간의 차이점에 대한 자세한 내용은 [지속 형 기능 버전](durable-functions-versions.md) 문서를 참조하십시오.

# <a name="javascript"></a>[자바 스크립트](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const transferDetails = context.df.getInput();

    yield context.df.callActivity("DebitAccount",
        {
            account = transferDetails.sourceAccount,
            amount = transferDetails.amount,
        }
    );

    try {
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.destinationAccount,
                amount = transferDetails.amount,
            }
        );
    }
    catch (error) {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.sourceAccount,
                amount = transferDetails.amount,
            }
        );
    }
});
```

---

첫 번째 **CreditAccount** 함수 호출이 실패하면 오케스트레이터 함수는 자금을 소스 계정에 다시 입금하여 보상합니다.

## <a name="automatic-retry-on-failure"></a>실패 시 자동 다시 시도

작업 함수 또는 하위 오케스트레이션 함수를 호출할 때 자동으로 다시 시도하는 정책을 지정할 수 있습니다. 다음 예제에서는 함수를 최대 3회 호출하려고 하고 각각의 다시 시도 간에 5초 동안 기다립니다.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("TimerOrchestratorWithRetry")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var retryOptions = new RetryOptions(
        firstRetryInterval: TimeSpan.FromSeconds(5),
        maxNumberOfAttempts: 3);

    await context.CallActivityWithRetryAsync("FlakyFunction", retryOptions, null);

    // ...
}
```

> [!NOTE]
> 이전 C# 예제는 지속 기능 2.x에 대 한 입니다. 지속 기능 1.x의 경우 `DurableOrchestrationContext` `IDurableOrchestrationContext`을 대신 사용해야 합니다. 버전 간의 차이점에 대한 자세한 내용은 [지속 형 기능 버전](durable-functions-versions.md) 문서를 참조하십시오.

# <a name="javascript"></a>[자바 스크립트](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const firstRetryIntervalInMilliseconds = 5000;
    const maxNumberOfAttempts = 3;

    const retryOptions = 
        new df.RetryOptions(firstRetryIntervalInMilliseconds, maxNumberOfAttempts);

    yield context.df.callActivityWithRetry("FlakyFunction", retryOptions);

    // ...
});
```

---

이전 예제의 활동 함수 호출은 자동 재시도 정책을 구성하기 위한 매개 변수를 사용합니다. 자동 재시도 정책을 사용자 지정하기 위한 몇 가지 옵션이 있습니다.

* **최대 시도 횟수**: 최대 다시 시도 횟수입니다.
* **첫 번째 다시 시도 간격**: 첫 번째 다시 시도를 수행할 때까지 기다리는 시간입니다.
* **백오프 계수**: 백오프의 증가율을 결정하는 데 사용되는 계수입니다. 기본값은 1입니다.
* **최대 다시 시도 간격**: 다시 시도 간에 기다리는 최대 시간입니다.
* **다시 시도 시간 제한**: 다시 시도하는 데 소요되는 최대 시간입니다. 기본 동작은 무기한으로 다시 시도하는 것입니다.
* **핸들**: 함수를 다시 시도해야 하는지 여부를 결정하기 위해 사용자 정의 콜백을 지정할 수 있습니다.

## <a name="function-timeouts"></a>함수 시간 제한

오케스트레이터 함수 내에서 함수 호출을 완료하는 데 시간이 너무 오래 걸리는 경우 함수 호출을 포기할 수 있습니다. 현재 이 작업을 제대로 수행하는 방법은 다음 예제와 같이 `Task.WhenAny`(.NET) 또는 `context.df.Task.any`(JavaScript)와 함께 `context.CreateTimer`(.NET) 또는 `context.df.createTimer`(JavaScript)를 사용하여 [지속성 타이머](durable-functions-timers.md)를 만드는 것입니다.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("TimerOrchestrator")]
public static async Task<bool> Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("FlakyFunction");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> 이전 C# 예제는 지속 기능 2.x에 대 한 입니다. 지속 기능 1.x의 경우 `DurableOrchestrationContext` `IDurableOrchestrationContext`을 대신 사용해야 합니다. 버전 간의 차이점에 대한 자세한 내용은 [지속 형 기능 버전](durable-functions-versions.md) 문서를 참조하십시오.

# <a name="javascript"></a>[자바 스크립트](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const deadline = moment.utc(context.df.currentUtcDateTime).add(30, "s");

    const activityTask = context.df.callActivity("FlakyFunction");
    const timeoutTask = context.df.createTimer(deadline.toDate());

    const winner = yield context.df.Task.any([activityTask, timeoutTask]);
    if (winner === activityTask) {
        // success case
        timeoutTask.cancel();
        return true;
    } else {
        // timeout case
        return false;
    }
});
```

---

> [!NOTE]
> 이 메커니즘은 실제로 진행 중인 작업 함수 실행을 종료하지 않습니다. 대신 단순히 오케스트레이터 함수를 사용하여 결과를 무시하고 계속 진행할 수 있습니다. 자세한 내용은 [타이머](durable-functions-timers.md#usage-for-timeout) 설명서를 참조하세요.

## <a name="unhandled-exceptions"></a>처리되지 않은 예외

처리되지 않은 예외로 인해 오케스트레이터 함수가 실패하면 예외 정보가 자세히 기록되고 인스턴스가 `Failed` 상태로 완료됩니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [영원한 오케스트레이션에 대해 알아보기](durable-functions-eternal-orchestrations.md)

> [!div class="nextstepaction"]
> [문제 진단](durable-functions-diagnostics.md)
