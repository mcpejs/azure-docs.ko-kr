---
title: 포함 파일
description: 포함 파일
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/07/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e84a77629026bb8885a48b8ed928699825f07115
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77111247"
---
| **공급 업체** | **디바이스 패밀리** | **펌웨어 버전** |
| --- | --- | --- |
|시스코 | ISR| IOS 15.1(미리 보기)|
|시스코 | ASA | ASA(*) RouteBased(IKEv2- BGP 없음), 9.8 미만 ASA |
|시스코 | ASA | ASA RouteBased(IKEv2- BGP 없음), 9.8 이상 ASA |
|Juniper | SRX_GA | 12.x|
|Juniper | SSG_GA | ScreenOS 6.2.x|
|Juniper | JSeries_GA | JunOS 12.x|
|Juniper | SRX | JunOS 12.x RouteBased BGP |
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased VTI|
|Ubiquiti| EdgeRouter| EdgeOS v1.10x RouteBased BGP|

> [!NOTE]
> ( * ) 필수: 좁은 AzureTraffic선택기(usePolicybasedTrafficSelectors 옵션 사용) 및 사용자 지정 Azure 정책(IKE/IPsec)
>
