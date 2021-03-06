---
title: 레지스트리 상태 확인
description: 빠른 진단 명령을 실행하여 로컬 Docker 구성 및 레지스트리 에 대한 연결을 포함하여 Azure 컨테이너 레지스트리를 사용할 때 일반적인 문제를 식별하는 방법을 알아봅니다.
ms.topic: article
ms.date: 07/02/2019
ms.openlocfilehash: ea4432c9e92c4a0380517e39678814e2d1cb3bfc
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74456425"
---
# <a name="check-the-health-of-an-azure-container-registry"></a>Azure 컨테이너 레지스트리의 상태 확인

Azure 컨테이너 레지스트리를 사용하는 경우 때때로 문제가 발생할 수 있습니다. 예를 들어 로컬 환경에서 Docker 문제가 있어 컨테이너 이미지를 가져오지 못할 수 있습니다. 또는 네트워크 문제로 인해 레지스트리에 연결하지 못할 수 있습니다. 

첫 번째 진단 단계로 [az acr 검사 상태][az-acr-check-health] 명령을 실행하여 환경의 상태와 선택적으로 대상 레지스트리에 대한 액세스에 대한 정보를 가져옵니다. 이 명령은 Azure CLI 버전 2.0.67 이상에서 사용할 수 있습니다. 설치 또는 업그레이드해야 하는 경우 [Azure CLI 설치][azure-cli]를 참조하십시오.

## <a name="run-az-acr-check-health"></a>az 체크 상태 실행

다음 예제는 `az acr check-health` 명령을 실행하는 다양한 방법을 보여 준다.

> [!NOTE]
> Azure Cloud Shell에서 명령을 실행하면 로컬 환경이 선택되지 않습니다. 그러나 대상 레지스트리에 대한 액세스를 확인할 수 있습니다.

### <a name="check-the-environment-only"></a>환경만 확인

로컬 Docker 데몬, CLI 버전 및 Helm 클라이언트 구성을 확인하려면 추가 매개 변수 없이 명령을 실행합니다.

```azurecli
az acr check-health
```

### <a name="check-the-environment-and-a-target-registry"></a>환경 및 대상 레지스트리 확인

레지스트리에 대한 액세스를 확인하고 로컬 환경 검사를 수행하려면 대상 레지스트리의 이름을 전달합니다. 예를 들어:

```azurecli
az acr check-health --name myregistry
```

## <a name="error-reporting"></a>오류 보고

명령은 정보를 표준 출력에 기록합니다. 문제가 발견되면 오류 코드와 설명을 제공합니다. 코드 및 가능한 해결 에 대한 자세한 내용은 [오류 참조](container-registry-health-error-reference.md)를 참조하십시오.

기본적으로 명령은 오류를 찾을 때마다 중지됩니다. 오류가 발견되더라도 모든 상태 확인에 대한 출력을 제공하므로 명령을 실행할 수도 있습니다. 다음 `--ignore-errors` 예제와 같이 매개 변수를 추가합니다.

```azurecli
# Check environment only
az acr check-health --ignore-errors

# Check environment and target registry
az acr check-health --name myregistry --ignore-errors
```      

샘플 출력:

```console
$ az acr check-health --name myregistry --ignore-errors --yes

Docker daemon status: available
Docker version: Docker version 18.09.2, build 6247962
Docker pull of 'mcr.microsoft.com/mcr/hello-world:latest' : OK
ACR CLI version: 2.2.9
Helm version:
Client: &version.Version{SemVer:"v2.14.1", GitCommit:"5270352a09c7e8b6e8c9593002a73535276507c0", GitTreeState:"clean"}
DNS lookup to myregistry.azurecr.io at IP 40.xxx.xxx.162 : OK
Challenge endpoint https://myregistry.azurecr.io/v2/ : OK
Fetch refresh token for registry 'myregistry.azurecr.io' : OK
Fetch access token for registry 'myregistry.azurecr.io' : OK
```  



## <a name="next-steps"></a>다음 단계

[az acr 확인 상태][az-acr-check-health] 명령에서 반환되는 오류 코드에 대한 자세한 내용은 상태 확인 오류 [참조를](container-registry-health-error-reference.md)참조하십시오.

자주 묻는 질문 및 Azure 컨테이너 레지스트리에 대한 기타 알려진 문제는 [FAQ를](container-registry-faq.md) 참조하십시오.





<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-check-health]: /cli/azure/acr#az-acr-check-health
