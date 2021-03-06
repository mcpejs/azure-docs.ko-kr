---
title: 그래픽 성능 문제 진단 원격 데스크톱 - Azure
description: 이 문서에서는 원격 데스크톱 프로토콜 세션에서 RemoteFX 그래픽 카운터를 사용하여 Windows 가상 데스크톱의 그래픽과 관련된 성능 문제를 진단하는 방법에 대해 설명합니다.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: troubleshooting
ms.date: 05/23/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 84cee86dbddff77f6142925eec01889cf793a466
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79127561"
---
# <a name="diagnose-graphics-performance-issues-in-remote-desktop"></a>원격 데스크톱의 그래픽 성능 문제 진단

원격 세션에서 환경 품질 문제를 진단하기 위해 성능 모니터의 RemoteFX 그래픽 섹션에서 카운터가 제공되었습니다. 이 도움말에서는 이러한 카운터를 사용하여 RDP(원격 데스크톱 프로토콜) 세션 중에 그래픽 관련 성능 병목 현상을 찾아내고 수정하는 데 도움이 됩니다.

## <a name="find-your-remote-session-name"></a>원격 세션 이름 찾기

그래픽 성능 카운터를 식별하려면 원격 세션 이름이 필요합니다. 이 섹션의 지침에 따라 각 카운터의 인스턴스를 식별합니다.

1. 원격 세션에서 Windows 명령 프롬프트를 엽니다.
2. **qwinsta** 명령을 실행하고 세션 이름을 찾습니다.
    - 세션이 VM(다중 세션 가상 머신)에서 호스팅되는 경우: 각 카운터의 인스턴스는 "rdp-tcp 37"과 같이 세션 이름을 접미사로 사용하는 동일한 번호로 접미사가 붙어 있습니다.
    - 가상 그래픽 처리 장치(vGPU)를 지원하는 VM에서 세션이 호스팅되는 경우: 각 카운터의 인스턴스는 VM이 아닌 서버에 저장됩니다. 카운터 인스턴스에는 세션 이름의 숫자 대신 "Win8 Enterprise VM"과 같은 VM 이름이 포함됩니다.

>[!NOTE]
> 카운터에는 RemoteFX가 있지만 vGPU 시나리오에도 원격 데스크톱 그래픽이 포함됩니다.

## <a name="access-performance-counters"></a>성능 카운터에 액세스

원격 세션 이름을 결정한 후 다음 지침에 따라 원격 세션에 대한 RemoteFX Graphics 성능 카운터를 수집합니다.

1. **관리 도구** > **성능 모니터** **시작을** > 선택합니다.
2. 성능 **모니터** 대화 상자에서 **모니터링 도구를**확장하고 **성능 모니터를**선택한 다음 **에 추가를**선택합니다.
3. 카운터 **추가** 대화 상자에서 사용 가능한 카운터 목록에서 RemoteFX 그래픽에 대한 섹션을 **확장합니다.**
4. 모니터링할 카운터를 선택합니다.
5. 선택한 **개체 목록의 인스턴스에서** 선택한 카운터에 대해 모니터링할 특정 인스턴스를 선택한 다음 **추가 를**선택합니다. 사용 가능한 모든 카운터 인스턴스를 선택하려면 **모든 인스턴스를**선택합니다.
6. 카운터를 추가한 후 **확인을**선택합니다.

선택한 성능 카운터가 성능 모니터 화면에 나타납니다.

>[!NOTE]
>호스트의 각 활성 세션에는 각 성능 카운터의 고유한 인스턴스가 있습니다.

## <a name="diagnose-issues"></a>문제 진단

그래픽 관련 성능 문제는 일반적으로 다음 네 가지 범주로 나뉩니다.

- 낮은 프레임 속도
- 랜덤 노점
- 높은 입력 대기 시간
- 잘못된 프레임 품질

### <a name="addressing-low-frame-rate-random-stalls-and-high-input-latency"></a>낮은 프레임 속도, 임의 스톨 및 높은 입력 대기 시간 해결

먼저 출력 프레임/세컨드 카운터를 확인합니다. 클라이언트에서 사용할 수 있는 프레임 수를 측정합니다. 이 값이 입력 프레임/세 번째 카운터보다 크면 프레임이 건너뜁니다. 병목 현상을 식별하려면 생략/두 번째 카운터를 사용합니다.

세 가지 유형의 프레임 건너뛰기/두 번째 카운터가 있습니다.

- 초당 프레임 건너뛰기(서버 리소스 부족)
- 프레임 건너뛰기/초(네트워크 리소스 부족)
- 프레임 건너뛰기/초(클라이언트 리소스 부족)

건너뛴/두 번째 카운터의 프레임에 대한 값이 높음은 카운터가 추적하는 리소스와 문제가 있음을 의미합니다. 예를 들어 클라이언트가 프레임을 디코딩하고 서버가 제공하는 것과 동일한 속도로 프레임을 표시하지 않으면 프레임 건너뛰기/초(클라이언트 리소스 부족) 카운터가 높습니다.

출력 프레임/세번째 카운터가 입력 프레임/세번째 카운터와 일치하지만 여전히 비정상적인 지연 또는 지연을 발견하면 평균 인코딩 시간이 원인일 수 있습니다. 인코딩은 단일 세션(vGPU) 시나리오의 서버와 다중 세션 시나리오의 VM에서 발생하는 동기 프로세스입니다. 평균 인코딩 시간은 33ms 미만이어야 합니다. 평균 인코딩 시간이 33ms 미만이지만 성능 문제가 있는 경우 사용 중인 앱 또는 운영 체제에 문제가 있을 수 있습니다.

앱 관련 문제 진단에 대한 자세한 내용은 [사용자 입력 지연 성능 카운터를](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/)참조하십시오.

RDP는 평균 인코딩 시간 33ms를 지원하므로 초당 최대 30프레임의 입력 프레임 속도를 지원합니다. 33ms는 지원되는 최대 프레임 속도입니다. 대부분의 경우, 소스에 의해 프레임이 RDP에 제공되는 빈도에 따라 사용자가 경험하는 프레임 속도는 더 낮아질 것이다. 예를 들어 비디오 시청과 같은 작업에는 초당 30프레임의 전체 입력 프레임 속도가 필요하지만, 문서를 자주 편집하지 않는 것과 같이 계산 집약적인 작업은 사용자의 경험 품질이 저하되지 않고 입력 프레임/초의 값이 훨씬 낮아집니다.

### <a name="addressing-poor-frame-quality"></a>열악한 프레임 품질 해결

프레임 품질 카운터를 사용하여 프레임 품질 문제를 진단합니다. 이 카운터는 출력 프레임의 품질을 소스 프레임 품질의 백분율로 표현합니다. 품질 손실은 RemoteFX로 인한 것일 수도, 아니면 그래픽 소스에 내재되어 있을 수 있습니다. RemoteFX에서 품질 이 저하된 경우 더 높은 충실도의 콘텐츠를 보낼 네트워크 또는 서버 리소스가 부족하여 문제가 발생할 수 있습니다.

## <a name="mitigation"></a>완화 방법

서버 리소스로 인해 병목 현상이 발생하는 경우 다음 방법 중 하나를 시도하여 성능을 향상시킵니다.

- 호스트당 세션 수를 줄입니다.
- 서버의 메모리를 늘리고 리소스를 계산합니다.
- 연결의 해상도를 삭제합니다.

네트워크 리소스로 인해 병목 현상이 발생하는 경우 다음 방법 중 하나를 시도하여 세션당 네트워크 가용성을 개선하십시오.

- 호스트당 세션 수를 줄입니다.
- 더 높은 대역폭 네트워크를 사용합니다.
- 연결의 해상도를 삭제합니다.

클라이언트 리소스가 병목 현상을 일으키는 경우 다음 방법 중 하나를 시도하여 성능을 향상시킵니다.

- 최신 원격 데스크톱 클라이언트를 설치합니다.
- 클라이언트 컴퓨터에서 메모리를 늘리고 리소스를 계산합니다.

> [!NOTE]
> 현재 소스 프레임/세컨드 카운터는 지원하지 않습니다. 지금은 소스 프레임/세컨드 카운터가 항상 0으로 표시됩니다.

## <a name="next-steps"></a>다음 단계

- GPU에 최적화된 Azure 가상 컴퓨터를 만들려면 [Windows 가상 데스크톱 환경에 대한 GPU(GPU) 가속 구성을](configure-vm-gpu.md)참조하십시오.
- 문제 해결 및 에스컬레이션 트랙에 대한 개요는 [문제 해결 개요, 피드백 및 지원을](troubleshoot-set-up-overview.md)참조하십시오.
- 서비스에 대한 자세한 내용은 [Windows 데스크톱 환경을](environment-setup.md)참조하십시오.
