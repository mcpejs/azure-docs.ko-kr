---
title: 프로덕션 환경에서 솔루션 배포 준비 - Azure IoT Edge
description: Azure IoT Edge 솔루션을 개발에서 프로덕션까지 수행하려면 적절한 인증서로 장치를 설정하고 향후 코드 업데이트를 위한 배포 계획을 수립하는 방법을 알아봅니다.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 4/02/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: dd24631f8e6b4f3f87438bf22654016dd7699950
ms.sourcegitcommit: bc738d2986f9d9601921baf9dded778853489b16
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/02/2020
ms.locfileid: "80618312"
---
# <a name="prepare-to-deploy-your-iot-edge-solution-in-production"></a>IoT Edge 솔루션을 프로덕션 단계에서 배포하도록 준비

IoT Edge 솔루션을 개발에서 프로덕션으로 전환할 준비가 되었으면 지속적인 성능 유지가 가능하도록 구성되었는지 확인합니다.

이 문서에 제공된 정보가 모두 동일하지는 않습니다. 우선 순위를 정하는 데 도움이 되도록 각 섹션의 시작 부분에 작업이 두 가지 범주로 구분되어 나열되어 있습니다. **중요**에 해당하는 작업은 프로덕션으로 전환하기 전에 완료해야 하며, **유용함**에 해당하는 작업은 참조용으로 알아두면 좋습니다.

## <a name="device-configuration"></a>디바이스 구성

IoT Edge 디바이스는 Raspberry Pi부터 노트북, 서버에서 실행되는 가상 머신에 이르기까지 모든 것을 망라할 수 있습니다. 물리적으로 또는 가상 연결을 통해 디바이스에 액세스할 수 있는 경우도 있고, 디바이스가 오랜 기간 동안 격리되었을 수도 있습니다. 어느 쪽이든 제대로 작동하도록 구성되었는지 확인해야 합니다.

* **중요**
  * 프로덕션 인증서 설치
  * 디바이스 관리 계획 준비
  * Moby를 컨테이너 엔진으로 사용

* **유용함**
  * 업스트림 프로토콜 선택

### <a name="install-production-certificates"></a>프로덕션 인증서 설치

프로덕션 단계의 모든 IoT Edge 디바이스에 디바이스 CA(인증 기관) 인증서가 설치되어 있어야 합니다. 설치된 CA 인증서는 config.yaml 파일에서 IoT Edge 런타임으로 선언됩니다. 개발 및 테스트 시나리오의 경우 IoT Edge 런타임은 config.yaml 파일에 인증서가 선언되지 않은 경우 임시 인증서를 만듭니다. 그러나 이러한 임시 인증서는 3개월 후에 만료되어 프로덕션 시나리오가 원활하게 진행될 수 없습니다.

디바이스 CA 인증서의 역할을 이해하려면 [Azure IoT Edge에서 인증서를 사용하는 방법](iot-edge-certs.md)을 참조하세요.

IoT Edge 장치에 인증서를 설치하고 config.yaml 파일에서 인증서를 참조하는 방법에 대한 자세한 내용은 [IoT Edge 장치에 프로덕션 인증서 설치를](how-to-manage-device-certificates.md)참조하십시오.

### <a name="have-a-device-management-plan"></a>디바이스 관리 계획 준비

디바이스를 프로덕션으로 전환하기 전에 향후 업데이트를 관리할 방법을 준비해 두어야 합니다. IoT Edge 디바이스의 경우 업데이트할 구성 요소 목록에 다음이 포함될 수 있습니다.

* 디바이스 펌웨어
* 운영 체제 라이브러리
* Moby와 같은 컨테이너 엔진
* IoT Edge 디먼
* CA 인증서

자세한 내용은 [IoT Edge 런타임 업데이트를](how-to-update-iot-edge.md)참조하십시오. 현재는 IoT Edge 디바이스에 물리적으로 또는 SSH를 통해 액세스해야만 IoT Edge 디먼을 업데이트할 수 있습니다. 업데이트할 장치가 많은 경우 업데이트 단계를 스크립트에 추가하거나 Ansible과 같은 자동화 도구를 사용하는 것이 좋습니다.

### <a name="use-moby-as-the-container-engine"></a>Moby를 컨테이너 엔진으로 사용

컨테이너 엔진은 모든 IoT Edge 장치의 필수 구성 조건입니다. 프로덕션 단계에서는 Moby 엔진만 지원됩니다. Docker 등 IoT Edge와 호환되는 다른 컨테이너 엔진은 개발 단계에서 사용할 수 있습니다. Moby 엔진은 Azure IoT Edge와 함께 사용할 때 재배포할 수 있으며, Microsoft에서 이 엔진에 대한 서비스를 제공합니다.

### <a name="choose-upstream-protocol"></a>업스트림 프로토콜 선택

IoT Edge 에이전트와 IoT Edge 허브 모두에 대해 IoT Hub에 대한 업스트림 통신을 위해 프로토콜(사용된 포트를 결정하는)을 구성할 수 있습니다. 기본 프로토콜은 AMQP이지만 네트워크 설정에 따라 변경할 수 있습니다.

두 개의 런타임 모듈에 모두 **UpstreamProtocol** 환경 변수가 있습니다. 변수에 유효한 값은 다음과 같습니다.

* MQTT
* AMQP
* MQTTWS
* AMQPWS

장치 자체의 config.yaml 파일에서 IoT Edge 에이전트에 대한 UpstreamProtocol 변수를 구성합니다. 예를 들어 IoT Edge 장치가 AMQP 포트를 차단하는 프록시 서버 뒤에 있는 경우 AmQP 를 통해 AMQP를 사용하도록 IoT Edge 에이전트를 구성하여 IoT Hub에 대한 초기 연결을 설정해야 할 수 있습니다.

IoT Edge 디바이스가 연결된 다음, 향후 배포에서 두 런타임 모듈에 대한 UpstreamProtocol 변수를 계속 구성해야 합니다. 이 프로세스의 예제는 [프록시 서버를 통해 통신하도록 IoT Edge 디바이스 구성](how-to-configure-proxy-support.md)에 나와 있습니다.

## <a name="deployment"></a>배포

* **유용함**
  * 업스트림 프로토콜의 일관성 유지
  * 시스템 모듈을 위한 호스트 스토리지 설정
  * IoT Edge 허브에서 사용하는 메모리 공간 감소
  * 모듈 이미지의 디버그 버전 사용 안 함

### <a name="be-consistent-with-upstream-protocol"></a>업스트림 프로토콜의 일관성 유지

IoT Edge 디바이스의 IoT Edge 에이전트를 기본 AMQP와 다른 프로토콜을 사용하도록 구성한 경우 향후 모든 배포에서 동일한 프로토콜을 선언해야 합니다. 예를 들어 IoT Edge 디바이스가 AMQP 포트를 차단하는 프록시 서버 뒤에 있는 경우 AMQPWS(WebSocket을 통한 AMQP)를 통해 연결하도록 디바이스를 구성했을 수 있습니다. 장치에 모듈을 배포할 때 IoT Edge 에이전트 및 IoT Edge 허브에 대해 동일한 AMQPWS 프로토콜을 구성하거나 기본 AMQP가 설정을 재정의하고 다시 연결하지 못하게 합니다.

IoT Edge 에이전트 및 IoT Edge 허브 모듈에 대한 UpstreamProtocol 환경 변수만 구성하면 됩니다. 모든 추가 모듈은 런타임 모듈에 설정된 프로토콜을 채택합니다.

이 프로세스의 예제는 [프록시 서버를 통해 통신하도록 IoT Edge 디바이스 구성](how-to-configure-proxy-support.md)에 나와 있습니다.

### <a name="set-up-host-storage-for-system-modules"></a>시스템 모듈을 위한 호스트 스토리지 설정

IoT Edge 허브 및 에이전트 모듈은 로컬 저장소를 사용하여 상태를 유지하고 모듈, 장치 및 클라우드 간에 메시징을 활성화합니다. 안정성과 성능을 향상하려면 시스템 모듈을 구성하여 호스트 파일 시스템의 저장소를 사용하도록 구성합니다.

자세한 내용은 [시스템 모듈에 대한 호스트 저장소를](how-to-access-host-storage-from-module.md)참조하십시오.

### <a name="reduce-memory-space-used-by-iot-edge-hub"></a>IoT Edge 허브에서 사용하는 메모리 공간 감소

제한된 메모리를 사용할 수 있는 제한된 장치를 배포하는 경우 보다 간소화된 용량으로 실행되고 디스크 공간을 적게 사용하도록 IoT Edge 허브를 구성할 수 있습니다. 그러나 이러한 구성은 IoT Edge 허브의 성능을 제한하므로 솔루션에 적합한 균형을 찾으십시오.

#### <a name="dont-optimize-for-performance-on-constrained-devices"></a>리소스가 제한된 디바이스에서 성능에 최적화하지 않음

IoT Edge 허브는 기본적으로 성능에 최적화되어 있으므로 큰 메모리 덩어리를 할당하려고 시도합니다. 이 구성으로 인해 Raspberry Pi와 같은 작은 디바이스에서 안정성 문제가 발생할 수 있습니다. 제한된 리소스가 있는 장치를 배포하는 경우 IoT Edge 허브에서 **OptimizeForPerformance** 환경 변수를 **false로** 설정할 수 있습니다.

**최적화ForPerformance가** **true로**설정된 경우 MQTT 프로토콜 헤드는 더 나은 성능을 가지고 있지만 더 많은 메모리를 할당하는 PooledByteBufferAllocator를 사용합니다. 할당자는 32비트 운영 체제 또는 메모리가 부족한 장치에서는 잘 작동하지 않습니다. 또한 성능에 최적화된 경우 RocksDb는 로컬 저장소 공급자로서의 역할에 더 많은 메모리를 할당합니다.

자세한 내용은 [리소스가 제한된 디바이스의 안정성 문제](troubleshoot.md#stability-issues-on-resource-constrained-devices)를 참조하세요.

#### <a name="disable-unused-protocols"></a>사용하지 않는 프로토콜 비활성화

IoT Edge 허브의 성능을 최적화하고 메모리 사용량을 줄이는 또 다른 방법은 솔루션에서 사용하지 않는 프로토콜에 대한 프로토콜 헤드를 끄는 것입니다.

프로토콜 헤드는 배포 매니페스트에서 IoT Edge 허브 모듈에 대한 부울 환경 변수를 설정하여 구성됩니다. 세 가지 변수는 다음과 같습니다.

* **amqpSettings__enabled**
* **mqttSettings__enabled**
* **httpSettings__enabled**

세 변수에는 모두 *두 개의 밑줄*이 있으며 true 또는 false로 설정할 수 있습니다.

#### <a name="reduce-storage-time-for-messages"></a>메시지의 스토리지 시간 축소

IoT Edge 허브 모듈은 어떤 이유로든 메시지를 IoT Hub로 전달할 수 없는 경우 일시적으로 저장합니다. IoT Edge 허브가 배달되지 않은 메시지에 대해 보유하는 기간(기간)을 구성한 후 만료되도록 할 수 있습니다. 장치에 메모리 문제가 있는 경우 IoT Edge 허브 모듈 트윈에서 **timeToLiveSecs** 값을 낮출 수 있습니다.

timeToLiveSecs 매개 변수의 기본값은 7200초로, 2시간입니다.

### <a name="do-not-use-debug-versions-of-module-images"></a>모듈 이미지의 디버그 버전 사용 안 함

테스트 시나리오에서 프로덕션 시나리오로 전환하는 경우 배포 매니페스트에서 디버그 구성을 제거해야 합니다. 배포 매니페스트의 모듈 이미지에 ** \.디버그** 접미사가 없는지 확인합니다. 디버깅을 위해 모듈의 포트를 노출하는 만들기 옵션을 추가한 경우 해당 만들기 옵션도 제거합니다.

## <a name="container-management"></a>컨테이너 관리

* **중요**
  * 컨테이너 레지스트리에 대한 액세스 관리
  * 태그를 사용하여 버전 관리

### <a name="manage-access-to-your-container-registry-with-a-service-principal"></a>서비스 주체를 사용하여 컨테이너 레지스트리에 대한 액세스 관리

프로덕션 IoT Edge 디바이스에 모듈을 배포하기 전에 외부 사용자가 컨테이너 이미지를 액세스하거나 변경할 수 없도록 컨테이너 레지스트리에 대한 액세스를 제어해야 합니다. 공용이 아닌 프라이빗 컨테이너 레지스트리를 사용하여 컨테이너 이미지를 관리합니다.

자습서 및 기타 설명서에서는 개발 머신에서 사용한 것과 동일한 컨테이너 레지스트리 자격 증명을 IoT Edge 디바이스에서 사용하도록 안내합니다. 이러한 지침은 테스트 및 개발 환경을 보다 쉽게 설정하도록 돕기 위한 것이며, 프로덕션 시나리오에서 따르면 안 됩니다. Azure 컨테이너 레지스트리는 응용 프로그램 또는 서비스가 IoT Edge 장치와 마찬가지로 자동화된 또는 무인 방식으로 컨테이너 이미지를 가져올 때 [서비스 주체로 인증할](../container-registry/container-registry-auth-service-principal.md) 것을 권장합니다.

서비스 주체를 만들려면 서비스 주체 만들기에 설명된 대로 두 [스크립트를 실행합니다.](../container-registry/container-registry-auth-aci.md#create-a-service-principal) 이러한 스크립트는 다음과 같은 작업을 수행합니다.

* 첫 번째 스크립트는 서비스 주체를 만듭니다. 서비스 주체 ID와 서비스 주체 암호를 출력합니다. 이러한 값을 레코드에 안전하게 저장합니다.

* 두 번째 스크립트는 서비스 주체에게 부여할 역할 할당을 생성하며, 필요한 경우 이후에 실행할 수 있습니다. `role` 매개 변수에 **acrPull** 사용자 역할을 적용하는 것이 좋습니다. 역할 목록은 [Azure 컨테이너 레지스트리 역할 및 사용 권한을 참조합니다.](../container-registry/container-registry-roles.md)

서비스 주체를 사용하여 인증하려면 첫 번째 스크립트에서 얻은 서비스 주체 ID와 암호를 제공합니다.

* 사용자 이름 또는 클라이언트 ID의 경우 서비스 주체 ID를 지정합니다.

* 암호 또는 클라이언트 보안 암호의 경우 서비스 주 암호를 지정합니다.

Azure CLI를 사용하여 컨테이너 인스턴스를 시작하는 예는 [서비스 주체를 사용하여 인증을](../container-registry/container-registry-auth-aci.md#authenticate-using-the-service-principal)참조하십시오.

### <a name="use-tags-to-manage-versions"></a>태그를 사용하여 버전 관리

태그는 docker 컨테이너의 버전을 구분하는 데 사용할 수 있는 docker 개념입니다. 태그는 컨테이너 리포지토리의 끝에 추가되는 **1.0** 등의 접미사입니다. 예를 들어 **mcr.microsoft.com/azureiotedge-agent:1.0**입니다. 태그는 변경 가능하며, 언제든지 다른 컨테이너를 가리키도록 변경할 수 있으므로 앞으로 모듈 이미지를 업데이트할 때 따를 규칙에 대해 팀이 합의해야 합니다.

또한 태그는 IoT Edge 디바이스에 업데이트를 적용하는 데 도움이 됩니다. 모듈의 업데이트된 버전을 컨테이너 레지스트리에 푸시할 때 태그를 증분합니다. 그런 다음, 태그가 증분된 새 배포를 디바이스에 푸시합니다. 컨테이너 엔진이 증분된 태그를 새 버전으로 인식하고 최신 모듈 버전을 디바이스로 끌어옵니다.

태그 규칙의 예제 및 IoT Edge에서 롤링 태그와 특정 태그를 사용하여 버전을 추적하는 방법을 알아보려면 [IoT Edge 런타임 업데이트](how-to-update-iot-edge.md#understand-iot-edge-tags)를 참조하세요.

## <a name="networking"></a>네트워킹

* **유용함**
  * 아웃바운드/인바운드 구성 검토
  * IoT 에지 장치에서 연결 허용
  * 프록시를 통한 통신 구성

### <a name="review-outboundinbound-configuration"></a>아웃바운드/인바운드 구성 검토

Azure IoT Hub과 IoT Edge 간의 통신 채널은 항상 아웃바운드로 구성됩니다. 대부분의 IoT Edge 시나리오에서는 세 개의 연결만 있으면 됩니다. 컨테이너 엔진은 모듈 이미지를 포함하는 컨테이너 레지스트리(한 개 또는 여러 개)에 연결해야 합니다. IoT Edge 런타임은 디바이스 구성 정보를 검색하고 메시지 및 원격 분석을 보내기 위해 IoT Hub에 연결해야 합니다. 또한 자동 프로비저닝을 사용하는 경우, IoT Edge 디먼은 디바이스 프로비저닝 서비스에 연결해야 합니다. 자세한 내용은 [방화벽 및 포트 구성 규칙](troubleshoot.md#firewall-and-port-configuration-rules-for-iot-edge-deployment)을 참조하세요.

### <a name="allow-connections-from-iot-edge-devices"></a>IoT 에지 장치에서 연결 허용

네트워킹 설정에서 IoT Edge 장치에서 만든 연결을 명시적으로 허용해야 하는 경우 다음 IoT Edge 구성 요소 목록을 검토하십시오.

* **IoT Edge 에이전트**는 WebSockets 등을 통해 IoT Hub에 대한 영구적 AMQP/MQTT 연결을 엽니다.
* **IoT Edge 허브**는 WebSockets 등을 통해 IoT Hub에 대한 단일 영구적 AMQP 연결이나 여러 MQTT 연결을 엽니다.
* **IoT Edge 디먼**은 IoT Hub에 대한 HTTPS 호출을 간헐적으로 수행합니다.

세 가지 경우 모두, DNS 이름이 \*.azure-devices.net 패턴과 일치합니다.

또한 **컨테이너 엔진**은 HTTPS를 통해 컨테이너 레지스트리를 호출합니다. IoT Edge 런타임 컨테이너 이미지를 검색하려면 DNS 이름이 mcr.microsoft.com입니다. 컨테이너 엔진은 배포에 구성된 대로 다른 레지스트리에 연결합니다.

다음 검사 목록은 방화벽 규칙의 시작점입니다.

   | URL(\* = 와일드카드) | 아웃바운드 TCP 포트 | 사용 |
   | ----- | ----- | ----- |
   | mcr.microsoft.com  | 443 | Microsoft 컨테이너 레지스트리 |
   | global.azure-devices-provisioning.net  | 443 | DPS 액세스(선택 사항) |
   | \*.azurecr.io | 443 | 개인 및 타사 컨테이너 레지스트리 |
   | \*.blob.core.windows.net | 443 | Blob 저장소에서 Azure 컨테이너 레지스트리 이미지 델타 다운로드 |
   | \*.azure-devices.net | 5671, 8883, 443 | IoT Hub 액세스 |
   | \*.docker.io  | 443 | 도커 허브 액세스(선택 사항) |

이러한 방화벽 규칙 중 일부는 Azure 컨테이너 레지스트리에서 상속됩니다. 자세한 내용은 [방화벽 뒤에 있는 Azure 컨테이너 레지스트리에 액세스하는 규칙 구성을](../container-registry/container-registry-firewall-access-rules.md)참조하십시오.

### <a name="configure-communication-through-a-proxy"></a>프록시를 통한 통신 구성

프록시 서버를 사용하는 네트워크에 사용자 디바이스를 배포하려는 경우 디바이스가 IoT Hub 및 컨테이너 레지스트리에 접근하기 위해 프록시를 통해 통신할 수 있어야 합니다. 자세한 내용은 [프록시 서버를 통해 통신하도록 IoT Edge 디바이스 구성](how-to-configure-proxy-support.md)을 참조하세요.

## <a name="solution-management"></a>솔루션 관리

* **유용함**
  * 로그 및 진단 설정
  * 테스트 및 CI/CD 파이프라인 고려

### <a name="set-up-logs-and-diagnostics"></a>로그 및 진단 설정

Linux에서 IoT Edge 데몬은 저널을 기본 로깅 드라이버로 사용합니다. 명령줄 도구 `journalctl`을 사용하여 디먼 로그를 쿼리할 수 있습니다. Windows에서 IoT Edge 디먼은 PowerShell 진단을 사용합니다. `Get-IoTEdgeLog`를 사용하여 디먼 로그를 쿼리합니다. IoT Edge 모듈은 JSON 드라이버를 로깅에 사용합니다.  

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

IoT Edge 배포를 테스트할 때는 일반적으로 사용자 디바이스에 액세스하여 로그를 검색하고 문제를 해결할 수 있습니다. 배포 시나리오에서는 해당 옵션이 없을 수 있습니다. 프로덕션에서 사용자 디바이스에 대한 정보를 수집하는 방법을 고려해 보세요. 한 가지 옵션은 다른 모듈에서 정보를 수집하여 클라우드로 보내는 로깅 모듈을 사용하는 것입니다. 로깅 모듈의 한 가지 예는 [logspout-loganalytics](https://github.com/veyalla/logspout-loganalytics)이며, 사용자가 직접 로깅 모듈을 설계할 수도 있습니다.

### <a name="place-limits-on-log-size"></a>로그 크기에 제한 배치

기본적으로 Moby 컨테이너 엔진은 컨테이너 로그 크기 제한을 설정하지 않습니다. 시간이 지남에 따라 장치가 로그로 가득 차고 디스크 공간이 부족할 수 있습니다. 이를 방지하기 위해 다음 옵션을 고려하십시오.

#### <a name="option-set-global-limits-that-apply-to-all-container-modules"></a>옵션: 모든 컨테이너 모듈에 적용되는 전역 제한 설정

컨테이너 엔진 로그 옵션에서 모든 컨테이너 로그 파일의 크기를 제한할 수 있습니다. 다음 예제에서는 로그 `json-file` 드라이버를 파일 의 크기와 수에 제한이 있는 (권장) 설정합니다.

```JSON
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    }
}
```

이 정보를 명명된 `daemon.json` 파일에 추가(또는 추가)하고 장치 플랫폼에 적합한 위치에 배치합니다.

| 플랫폼 | 위치 |
| -------- | -------- |
| Linux | `/etc/docker/` |
| Windows | `C:\ProgramData\iotedge-moby\config\` |

변경 사항이 적용되려면 컨테이너 엔진을 다시 시작해야 합니다.

#### <a name="option-adjust-log-settings-for-each-container-module"></a>옵션: 각 컨테이너 모듈에 대한 로그 설정 조정

**당신은 만들기에서** 그렇게 할 수 있습니다각 모듈의 옵션. 예를 들어:

```yml
"createOptions": {
    "HostConfig": {
        "LogConfig": {
            "Type": "json-file",
            "Config": {
                "max-size": "10m",
                "max-file": "3"
            }
        }
    }
}
```

#### <a name="additional-options-on-linux-systems"></a>리눅스 시스템에 추가 옵션

* 기본 로깅 드라이버로 설정하여 `systemd` [journal](https://docs.docker.com/config/containers/logging/journald/) `journald` 저널에 로그를 보낼 컨테이너 엔진을 구성합니다.

* 주기적으로 logrotate 도구를 설치하여 장치에서 이전 로그를 제거합니다. 다음 파일 사양을 사용합니다.

   ```txt
   /var/lib/docker/containers/*/*-json.log{
       copytruncate
       daily
       rotate7
       delaycompress
       compress
       notifempty
       missingok
   }
   ```

### <a name="consider-tests-and-cicd-pipelines"></a>테스트 및 CI/CD 파이프라인 고려

가장 효율적인 IoT Edge 배포 시나리오를 위해 프로덕션 배포를 테스트 및 CI/CD 파이프라인에 통합하는 것이 좋습니다. Azure IoT Edge는 Azure DevOps를 포함하여 여러 CI/CD 플랫폼을 지원합니다. 자세한 내용은 [Azure IoT Edge 연속 통합 및 지속적인 배포](how-to-ci-cd.md)를 참조하세요.

## <a name="next-steps"></a>다음 단계

* [IoT Edge 자동 배포](module-deployment-monitoring.md)에 대해 자세히 알아보세요.
* IoT Edge에서 [연속 통합 및 지속적인 배포](how-to-ci-cd.md)를 지원하는 방법을 확인합니다.
