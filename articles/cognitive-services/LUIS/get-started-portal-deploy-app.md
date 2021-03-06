---
title: '빠른 시작: LUIS 포털에서 앱 배포'
titleSuffix: Azure Cognitive Services
description: 이 빠른 시작에서는 예측 엔드포인트 리소스를 만들고, 리소스를 할당하고, 앱을 학습시키고 게시하여 앱을 배포하는 방법을 보여 줍니다.
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: quickstart
ms.date: 01/27/2020
ms.author: diberry
ms.openlocfilehash: 0ee2b33aa3388b3cb99aa42c338ded800c9679a4
ms.sourcegitcommit: fe6c9a35e75da8a0ec8cea979f9dec81ce308c0e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/26/2020
ms.locfileid: "79221184"
---
# <a name="quickstart-deploy-an-app-in-the-luis-portal"></a>빠른 시작: LUIS 포털에서 앱 배포

LUIS 앱이 챗봇 같은 클라이언트 애플리케이션으로 발화 예측을 반환할 준비가 되면 예측 엔드포인트에 앱을 배포해야 합니다.

이 빠른 시작에서는 애플리케이션을 배포하는 것에 대해 알아봅니다. 예측 엔드포인트 리소스를 만들고, 앱에 리소스를 할당하고, 앱을 학습시키고, 앱을 게시합니다.

## <a name="prerequisites"></a>사전 요구 사항

* [Azure 구독](https://azure.microsoft.com/free)을 가져옵니다.
* [이전 포털 빠른 시작](get-started-portal-build-app.md)을 완료하거나 [앱을 다운로드하고 가져옵니다](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/documentation-samples/quickstarts/in-portal/build-portal-app.json).
* 이전의 Azure 리소스 인증을 받은 앱이 있는 경우 [Azure 리소스로 마이그레이션](luis-migration-authoring.md)합니다. 이메일 인증이 적용되는 경우 일부 포털 페이지가 다르게 보입니다.

## <a name="create-the-endpoint-resource"></a>엔드포인트 리소스 만들기

예측 엔드포인트 리소스를 Azure Portal에서 만듭니다. 이 리소스는 엔드포인트 예측 쿼리에만 사용해야 합니다. 앱을 변경하는 작업에는 이 리소스를 사용하지 마세요.

1. [Azure Portal ](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne)에서 로그인하고 리소스를 만듭니다.

1. 다음 설정으로 구독을 구성합니다.

   |설정|값|목적|
   |--|--|--|
   |속성|`my-luis-resource`|Azure 리소스의 이름입니다. LUIS 포털에서 앱에 리소스를 할당할 때 이 이름이 필요합니다.|
   |Subscription|사용자의 구독|계정과 연결된 구독 중 하나를 선택합니다.|
   |Resource group|`my-resource-group`|모든 인식 서비스 리소스에 대한 새 리소스 그룹을 만듭니다. 리소스 사용을 마친 후에는 리소스 그룹을 삭제하여 구독을 정리할 수 있습니다. |
   |작성 위치|**미국 서부**|작성할 Azure 지역입니다.|
   |작성 가격 책정 계층|**F0**|작성할 기본 가격 책정 계층입니다.|
   |런타임 위치|**미국 서부**|예측 엔드포인트 쿼리를 위한 Azure 지역입니다.|
   |런타임 가격 책정 계층|**S0**|이 가격 책정 계층에서는 트래픽이 많은 웹 사이트를 제공합니다.|
   | | | |


   ![Azure API 선택](./media/luis-how-to-azure-subscription/create-resource-in-azure.png)

1. **만들기**를 선택하여 Azure 리소스를 만듭니다.

   다음 섹션에서는 LUIS 포털에서 이 새 리소스를 LUIS 앱에 연결하는 방법을 보여줍니다.

## <a name="assign-the-resource-key-to-the-luis-app-in-the-luis-portal"></a>LUIS 포털에서 LUIS 앱에 리소스 키 할당

LUIS에 대한 새 리소스를 만들 때마다 LUIS 앱에 리소스를 할당해야 합니다. 리소스가 할당되면 새 리소스를 만들지 않는 이상 이 단계를 다시 수행할 필요가 없습니다. 앱의 영역을 확장하거나 더 많은 예측 쿼리를 지원하기 위해 새 리소스를 만들어야 하는 경우가 있습니다.

1. [LUIS 포털 미리 보기](https://preview.luis.ai)에 로그인하고 앱 목록에서 **myEnglishApp** 앱을 선택합니다.

1. 오른쪽 상단 메뉴에서 **관리**를 선택한 다음, **Azure Resources**를 선택합니다.

1. LUIS를 추가하려면 **예측 리소스 추가**를 선택합니다.

    ![LUIS 예측 리소스를 추가하기 위해 예측 리소스 추가 선택](./media/get-started-portal-deploy-app/azure-resources-add-prediction-resource.png)

1. 테넌트, 구독 및 리소스 이름을 선택합니다. **리소스 할당**을 선택합니다.

   ![앱에 리소스 할당](./media/get-started-portal-deploy-app/assign-resource.png)

1. 작성 키를 앱에 추가하려면 동일한 단계를 수행합니다.

1. 테이블에서 새 예측 리소스에 대한 새 행을 찾고, 엔드포인트 URL을 복사합니다. 예측을 위한 LUIS API 엔드포인트에 `HTTP GET`을 요청할 수 있도록 올바르게 구성되어 있습니다.

> [!TIP]
> 활성 학습을 사용하여 LUIS 앱을 개선하려면 **쿼리 매개 변수 변경**을 선택하고 **로그 저장**을 선택합니다. 이 작업은 `log=true` 쿼리 문자열 매개 변수를 추가하여 예제 URL을 변경합니다. 런타임 엔드포인트에 예측 쿼리를 만들 때 변경된 예제 쿼리 URL을 복사하여 사용합니다.

## <a name="train-the-app"></a>앱 학습

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="publish-the-app-to-the-prediction-endpoint"></a>예측 엔드포인트에 앱 게시

[!INCLUDE [LUIS How to Train steps](includes/howto-publish.md)]

## <a name="prediction-endpoint-request"></a>예측 엔드포인트 요청

미리 보기 포털에서 URL 끝에 있는 `query=`는 사용자의 발화가 GET 요청에 추가되는 위치입니다. 이전 빠른 시작의 끝부분에 사용한 것과 동일한 사용자 발화를 `query=` 뒤에 입력합니다.

```Is there a form named hrf-234098```

쿼리 문자열에 다음 쌍이 포함되어 있는지 확인합니다.

* `show-all-intents=true`
* `verbose=true`

브라우저에 응답이 표시됩니다.

```JSON
{
    "query": "Is there a form named hrf-234098",
    "prediction": {
        "topIntent": "FindForm",
        "intents": {
            "FindForm": {
                "score": 0.9768753
            },
            "None": {
                "score": 0.0216071177
            }
        },
        "entities": {
            "Human Resources Form Number": [
                "hrf-234098"
            ],
            "$instance": {
                "Human Resources Form Number": [
                    {
                        "type": "Human Resources Form Number",
                        "text": "hrf-234098",
                        "startIndex": 22,
                        "length": 10,
                        "modelTypeId": 8,
                        "modelType": "Regex Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

이와 동일한 수준의 정보를 테스트 창에 표시하려면 앱을 게시해야 합니다. 앱을 게시한 후 테스트 창에서 **게시된 버전과 비교**를 선택합니다. 게시된 테스트 창의 **JSON 보기 표시**를 사용하여 이전 단계와 동일한 JSON을 볼 수 있습니다. 이렇게 하면 사용 중인 현재 앱의 변경 내용을 엔드포인트에 게시된 앱과 비교할 수 있습니다.

[![현재 편집 중인 앱 버전과 게시된 앱 버전 비교](./media/get-started-portal-deploy-app/compare-test-pane.png)](./media/get-started-portal-deploy-app/compare-test-pane.png#lightbox)

## <a name="clean-up-resources"></a>리소스 정리

이 빠른 시작을 마쳤으면 위쪽 탐색 메뉴에서 **내 앱**을 선택합니다. 목록에서 앱의 확인란을 선택한 다음, 목록 위의 상황에 맞는 도구 모음에서 **삭제**를 선택합니다.

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"]
> [일반적인 의도 및 엔터티 식별](luis-tutorial-prebuilt-intents-entities.md)
