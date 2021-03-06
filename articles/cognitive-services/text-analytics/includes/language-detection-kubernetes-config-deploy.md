---
title: 언어 감지 Kubernetes 구성 및 단계 배포
titleSuffix: Azure Cognitive Services
description: 언어 감지 Kubernetes 구성 및 단계 배포
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: dc344bd6e4927d39b72ba9af3ae2eeb61d9cec95
ms.sourcegitcommit: 2d7910337e66bbf4bd8ad47390c625f13551510b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/08/2020
ms.locfileid: "80877839"
---
### <a name="deploy-the-language-detection-container-to-an-aks-cluster"></a>AKS 클러스터에 언어 검색 컨테이너 배포

1. Azure CLI를 열고 Azure에 로그인합니다.

    ```azurecli
    az login
    ```

1. AKS 클러스터에 로그인합니다. 적절한 `your-cluster-name` `your-resource-group` 값으로 바꿉을 바꿉습니다.

    ```azurecli
    az aks get-credentials -n your-cluster-name -g -your-resource-group
    ```

    이 명령을 실행한 후 다음과 유사한 메시지를 보고합니다.

    ```output
    Merged "your-cluster-name" as current context in /home/username/.kube/config
    ```

    > [!WARNING]
    > Azure 계정에서 여러 구독을 사용할 수 있고 `az aks get-credentials` 명령이 오류와 함께 반환되는 경우 일반적인 문제는 잘못된 구독을 사용하고 있다는 것입니다. Azure CLI 세션의 컨텍스트를 설정하여 리소스를 만든 것과 동일한 구독을 사용하고 다시 시도합니다.
    > ```azurecli
    >  az account set -s subscription-id
    > ```

1. 선택한 텍스트 편집기열기. 이 예제에서는 시각적 스튜디오 코드를 사용합니다.

    ```console
    code .
    ```

1. 텍스트 편집기 내에서 *language.yaml라는*새 파일을 만들고 다음 YAML을 붙여 넣습니다. 자신의 정보를 `billing/value` 교체하고 `apikey/value` 사용자 고유의 정보로 교체해야 합니다.

    ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: language
    spec:
      template:
        metadata:
          labels:
            app: language-app
        spec:
          containers:
          - name: language
            image: mcr.microsoft.com/azure-cognitive-services/language
            ports:
            - containerPort: 5000
            resources:
              requests:
                memory: 2Gi
                cpu: 1
              limits:
                memory: 4Gi
                cpu: 1
            env:
            - name: EULA
              value: "accept"
            - name: billing
              value: # {ENDPOINT_URI}
            - name: apikey
              value: # {API_KEY}
     
    --- 
    apiVersion: v1
    kind: Service
    metadata:
      name: language
    spec:
      type: LoadBalancer
      ports:
      - port: 5000
      selector:
        app: language-app
    ```

1. 파일을 저장하고 텍스트 편집기닫습니다.
1. *language.yaml* `apply` 파일을 대상으로 Kubernetes 명령을 실행합니다.

    ```console
    kubectl apply -f language.yaml
    ```

    명령이 배포 구성을 성공적으로 적용하면 메시지는 다음 출력과 유사하게 나타납니다.

    ```output
    deployment.apps "language" created
    service "language" created
    ```
1. 포드가 배포되었는지 확인합니다.

    ```console
    kubectl get pods
    ```

    포드의 실행 상태에 대한 출력:

    ```output
    NAME                         READY     STATUS    RESTARTS   AGE
    language-5c9ccdf575-mf6k5   1/1       Running   0          1m
    ```

1. 서비스를 사용할 수 있는지 확인하고 IP 주소를 가져옵니다.

    ```console
    kubectl get services
    ```

    포드에서 *언어* 서비스의 실행 상태에 대한 출력:

    ```output
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          2m
    language     LoadBalancer   10.0.100.64   168.61.156.180   5000:31234/TCP   2m
    ```
