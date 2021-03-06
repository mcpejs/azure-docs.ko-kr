---
title: Azure 서비스 버스 - 메시지 수
description: Azure 리소스 관리자 및 Azure 서비스 버스 NamespaceManager API를 사용하여 큐 및 구독에 보유된 메시지 수를 검색합니다.
services: service-bus-messaging
documentationcenter: ''
author: axisc
manager: timlt
editor: spelluru
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/08/2020
ms.author: aschhab
ms.openlocfilehash: 8020b12ca892fbf7dec6fed6259526d958fb110f
ms.sourcegitcommit: df8b2c04ae4fc466b9875c7a2520da14beace222
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/08/2020
ms.locfileid: "80891767"
---
# <a name="message-counters"></a>메시지 카운터

Azure Resource Manager와 .NET Framework SDK의 Service Bus [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) API를 사용하여 큐 및 구독에 보관된 메시지 수를 검색할 수 있습니다.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

PowerShell로 다음과 같이 수를 얻을 수 있습니다.

```powershell
(Get-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue).CountDetails
```

## <a name="message-count-details"></a>메시지 수 세부 정보

활성 메시지 수를 알면 큐가 현재 배포된 것보다 더 많은 리소스를 처리해야 하는 백로그를 빌드하는지 여부를 파악하는 데 유용합니다. 다음 카운터 세부 정보는 [MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) 클래스에 있습니다.

-   [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ActiveMessageCount): 활성 상태에 있고 배달 준비가 된 큐 또는 구독의 메시지입니다.
-   [DeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.deadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_DeadLetterMessageCount): 배달하지 못한 편지 큐의 메시지입니다.
-   [ScheduledMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.scheduledmessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ScheduledMessageCount): 예약된 상태의 메시지입니다.
-   [TransferDeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transferdeadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferDeadLetterMessageCount): 다른 큐 또는 토픽으로 전송하지 못하고 배달하지 못한 편지 큐로 이동된 메시지입니다.
-   [TransferMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transfermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferMessageCount): 다른 큐 또는 토픽으로 전송 대기중인 메시지입니다.

애플리케이션이 큐의 길이에 따라 리소스 크기를 조정하려는 경우 정교한 속도로 조정해야 합니다. 메시지 카운터를 확보하는 작업은 메시지 broker 내부에 비용이 많이 소요되며 빈번하게 직접 실행하면 엔터티 성능에 부정적인 영향을 줍니다.

> [!NOTE]
> Service Bus 토픽으로 전송되는 메시지는 해당 토픽의 구독으로 전달됩니다. 따라서 해당 메시지가 구독으로 성공적으로 전달되므로 토픽 자체의 활성 메시지 수는 0입니다. 구독에서 메시지 수를 확인하고 메시지 수가 0보다 큰지 확인합니다. 구독에서 메시지가 표시되더라도 실제로 는 토픽이 소유한 저장소에 저장됩니다. 

구독을 살펴보면 메시지 수가 0이 아닌 메시지 수가 됩니다(이 전체 엔터티에 대해 최대 323MB의 공간이 추가됩니다).

## <a name="next-steps"></a>다음 단계

Service Bus 메시징에 대해 자세히 알아보려면 다음 항목을 참조하세요.

* [Service Bus 큐, 토픽 및 구독](service-bus-queues-topics-subscriptions.md)
* [Service Bus 큐 시작](service-bus-dotnet-get-started-with-queues.md)
* [Service Bus 토픽 및 구독을 사용하는 방법](service-bus-dotnet-how-to-use-topics-subscriptions.md)
