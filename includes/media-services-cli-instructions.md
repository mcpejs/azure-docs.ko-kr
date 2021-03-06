---
title: 포함 파일
description: 포함 파일
services: media-services
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 01/28/2019
ms.author: juliako
ms.custom: include file
ms.openlocfilehash: 5ebbac39c8850737ea6f9ef333e45d305a520655
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79461217"
---
## <a name="use-cli-shell"></a>CLI Shell 사용

[Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview?view=azure-cli-latest)을 사용하여 CLI 명령을 실행하는 것이 좋습니다. **Cloud Shell**은 이 문서의 단계를 실행하는 데 무료로 사용할 수 있는 대화형 셸입니다. 계정에서 사용할 수 있도록 공용 Azure 도구가 Cloud Shell에 사전 설치 및 구성되어 있습니다. 작업 방식에 가장 적합한 셸 환경을 유연하게 선택할 수 있습니다. Linux 사용자는 Bash 환경을 선택할 수 있으며, Windows 사용자는 PowerShell을 선택할 수 있습니다.

CLI를 로컬로 설치할 수도 있습니다. 플랫폼과 관련된 지침은 [Azure CLI 설치](https://docs.microsoft.com/cli/azure/install-azure-cli)를 참조하세요.

### <a name="sign-in"></a>로그인

CLI를 로컬로 설치하려면 Azure에 로그인해야 합니다. Azure Cloud Shell에서는 이 단계가 필요 없습니다. `az login` 명령을 사용하여 로그인합니다.

CLI가 기본 브라우저를 열 수 있는 경우, 그렇게 하고 로그인 페이지를 로드합니다. 그렇지 않은 경우, 브라우저 페이지를 열고 명령줄의 지침에 따라 브라우저에서 https://aka.ms/devicelogin 으로 이동한 후 권한 부여 코드를 입력합니다.

### <a name="specify-location-of-files"></a>파일의 위치 지정

여러 Media Services CLI 명령을 사용하면 파일 이름으로 매개 변수를 전달할 수 있습니다. **Cloud Shell**을 사용하는 경우 파일을 클라우드 드라이브에 업로드할 수 있습니다(Bash 또는 PowerShell 사용). 

![파일 업로드]

로컬 CLI 또는 **Cloud Shell**을 사용하는 경우 사용 중인 OS 또는 Cloud Shell(Bash 또는 PowerShell)에 따라 파일 경로를 지정해야 합니다. 다음은 몇 가지 예입니다.

상대 파일 경로(모든 OS)

* `@"mytestfile.json"`
* `@"../mytestfile.json"`

Linux/Mac 및 Windows OS의 절대 파일 경로

* `@ "/usr/home/mytestfile.json"`
*    `@"c:\tmp\user\mytestfile.json"`

명령이 파일 경로를 묻는 경우 `{file}`을 사용하세요. `az ams transform create -a amsaccount -g resourceGroup -n custom --preset .\customPreset.json`)을 입력합니다. <br/> 명령이 특정 파일을 로드하려는 경우 `@{file}`을 사용하세요. `az ams account-filter create -a amsaccount -g resourceGroup -n filterName --tracks @tracks.json`)을 입력합니다.

[파일 업로드]: ./media/media-services-cli/upload-download-files.png
