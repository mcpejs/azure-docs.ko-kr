---
title: 고가용성 - Azure Dedicated HSM | Microsoft Docs
description: Azure Dedicated HSM 고가용성 예제 및 기본적인 고려 사항
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
ms.custom: mvc, seodec18
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/27/2019
ms.author: mbaldwin
ms.openlocfilehash: 536ef62acad900090924598edfa45450b2a8c951
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "70882259"
---
# <a name="azure-dedicated-hsm-high-availability"></a>Azure Dedicated HSM 고가용성

Azure Dedicated HSM은 Microsoft의 고가용성 데이터 센터에서 기반이 됩니다. 그러나 모든 고가용성 데이터 센터는 지역 수준 오류인 극단적인 상황 및 지역화된 오류에 취약합니다. Microsoft에서는 여러 디바이스의 프로비전으로 인해 해당 디바이스가 단일 랙을 공유하지 못하도록 지역 내의 다른 데이터 센터에 HSM 디바이스를 배포합니다. Gemalto HA 그룹 기능을 사용하여 지역의 데이터 센터에서 이러한 HSM을 페어링하면 추가 수준의 고가용성을 얻을 수 있습니다. 재해 복구 상황에서 지역 장애 조치(failover)를 해결하기 위해 지역 간에 디바이스를 연결할 수도 있습니다. 이 다계층 고가용성 구성을 사용하면 모든 디바이스 오류를 자동으로 해결하여 애플리케이션이 작업을 유지합니다. 또한 모든 데이터 센터에는 현장에 예비 디바이스 및 구성 요소가 있으므로 시기 적절하게 실패한 모든 디바이스를 바꿀 수 있습니다.

## <a name="high-availability-example"></a>고가용성 예제

소프트웨어 수준에서 고가용성에 대한 HSM 디바이스를 구성하는 방법에 대한 내용은 'Gemalto Luna Network HSM 관리 가이드'에 있습니다. 이 문서는 [젬알토 HSM 페이지에서](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/)확인할 수 있습니다.

다음 다이어그램에서는 고가용성 아키텍처를 보여 줍니다. 이 아키텍처는 지역에서 여러 디바이스 및 별도 지역에서 연결된 여러 디바이스를 사용합니다. 또한 최소 4개의 HSM 디바이스 및 가상 네트워킹 구성 요소도 사용합니다.

![고가용성 다이어그램](media/high-availability/high-availability.png)

## <a name="next-steps"></a>다음 단계

고가용성 및 보안과 같은 서비스의 모든 주요 개념은 디바이스 프로비전 및 애플리케이션 디자인 또는 배포 전에 제대로 이해하는 것이 좋습니다.
추가 개념 수준 항목:

* [배포 아키텍처](deployment-architecture.md)
* [물리적 보안](physical-security.md)
* [네트워킹](networking.md)
* [지원 가능성](supportability.md)
* <bpt i="1000001" x="1000001" type="formatting">{b&gt;</bpt>모니터링<ept i="1000001">&lt;b}</ept>

고가용성을 위한 HSM 디바이스를 구성하는 방법에 대한 자세한 내용은 Gemalto 고객 지원 포털 관리자 가이드 및 섹션 6을 참조하세요.
