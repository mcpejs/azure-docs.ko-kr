---
title: 포함 파일
titleSuffix: Azure
description: 포함 파일
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 6f5d2dc30ac0f6316587fa0836b87cbd4efc0a8b
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75774394"
---
PowerShell cmdlet **Get-AzPeeringLocation은** 이후 단계에서 사용할 필수 `Kind`매개 변수를 사용하여 피어링 위치 목록을 반환합니다.

```powershell
Get-AzPeeringLocation -Kind "Exchange"
```

Exchange 피어링 위치에는 다음 필드가 포함됩니다.
* 교환 이름
* 피어링위치
* 국가
* 피어링DB시설ID
* 피어링DB시설링크
* 마이크로 소프트IPv4주소
* 마이크로소프트IPv6주소

[PeeringDB](https://wwww.peeringdb.com)를 참조하여 원하는 피어링 시설에 있는지 확인합니다.

다음은 시애틀을 피어링 위치로 사용하여 피어링을 만드는 방법을 보여 주는 예제입니다.

```powershell
$exchangeLocations = Get-AzPeeringLocation -Kind Exchange
$exchangeLocation = $exchangeLocations | where {$_.PeeringLocation -eq "Seattle"}

#check the location metadata
$exchangeLocation

ExchangeName          : Columbia IX
PeeringLocation       : Seattle
Country               : US
PeeringDBFacilityId   : 99999
PeeringDBFacilityLink : https://www.peeringdb.com/ix/99999
MicrosoftIPv4Address  : 10.12.97.129
MicrosoftIPv6Address  :

ExchangeName          : Equinix Seattle
PeeringLocation       : Seattle
Country               : US
PeeringDBFacilityId   : 11
PeeringDBFacilityLink : https://www.peeringdb.com/ix/11
MicrosoftIPv4Address  : 198.32.134.152
MicrosoftIPv6Address  : 2001:504:12::15

...

```