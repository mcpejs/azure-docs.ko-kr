---
author: dlepow
ms.service: container-instances
ms.topic: include
ms.date: 02/13/2019
ms.author: danlep
ms.openlocfilehash: 33a82a55b03cde79d2d80826041ee6d43565476a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "80334810"
---
| 리소스 | 제한 |
| --- | :--- |
| [구독당](../articles/billing-buy-sign-up-azure-subscription.md) 지역별 표준 sku 컨테이너 그룹 | 100<sup>1</sup> |
| [구독당](../articles/billing-buy-sign-up-azure-subscription.md) 지역별 전용 sku 컨테이너 그룹 | 0<sup>1</sup> |
| 컨테이너 그룹당 컨테이너 개수 | 60 |
| 컨테이너 그룹당 볼륨 개수 | 20 |
| IP당 포트 수 | 5 |
| 컨테이너 인스턴스 로그 크기 - 실행 중인 인스턴스 | 4MB |
| 컨테이너 인스턴스 로그 크기 - 중지된 인스턴스 | 16KB 또는 1000개 줄 |
| 시간당 컨테이너 생성 수 |300<sup>1</sup> |
| 5분당 컨테이너 생성 수 | 100<sup>1</sup> |
| 시간당 컨테이너 삭제 수 | 300<sup>1</sup> |
| 5분당 컨테이너 삭제 수 | 100<sup>1</sup> |


<sup>1</sup> 제한 증가를 요청하려면 [Azure 지원 요청][azure-support]을 만듭니다.<br />

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
