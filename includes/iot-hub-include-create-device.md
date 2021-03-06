---
title: 포함 파일
description: 포함 파일
services: iot-hub
author: robinsh
ms.service: iot-hub
ms.topic: include
ms.date: 11/06/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 7b022f71e197c5695876f2049ee376c3616afc6d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "70049056"
---
<!-- put the ## header in the file that includes this file -->

이 섹션에서는 IoT 허브의 ID 레지스트리에 디바이스 ID를 만듭니다. 장치는 ID 레지스트리에 항목이 없으면 허브에 연결할 수 없습니다. 자세한 내용은 [IoT Hub 개발자 가이드를](../articles/iot-hub/iot-hub-devguide-identity-registry.md#identity-registry-operations)참조하십시오.

1. IoT 허브 탐색 메뉴에서 **IoT 장치를**연 다음 **새로를** 선택하여 IoT 허브에 장치를 추가합니다.

    ![포털에서 디바이스 ID 만들기](./media/iot-hub-include-create-device/create-identity-portal-vs2019.png)

1. **장치 만들기에서** **myDeviceId**와 같은 새 장치의 이름을 제공하고 **저장을**선택합니다. 이 작업은 IoT 허브에 대한 장치 ID를 만듭니다.

   ![새 디바이스 추가](./media/iot-hub-include-create-device/create-a-device-vs2019.png)

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

1. 디바이스가 만들어진 후 **IoT 디바이스** 창의 목록에서 디바이스를 엽니다. 나중에 사용할 **기본 연결 문자열을** 복사합니다.

    ![디바이스 연결 문자열](./media/iot-hub-include-create-device/device-details-vs2019.png)

> [!NOTE]
> IoT Hub ID 레지스트리는 디바이스 ID만 저장하여 IoT Hub에 보안 액세스를 사용합니다. 보안 자격 증명으로 사용하기 위해 디바이스 ID 및 키와 개별 디바이스에 대해 액세스하지 못하도록 설정할 수 있는 사용/사용 안 함 플래그를 저장합니다. 애플리케이션이 다른 디바이스별 메타데이터를 저장해야 할 경우 애플리케이션별 스토리지를 사용해야 합니다. 자세한 내용은 [IoT Hub 개발자 가이드](../articles/iot-hub/iot-hub-devguide-identity-registry.md)를 참조하세요.
