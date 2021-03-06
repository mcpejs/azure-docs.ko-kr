---
title: '의사 결정 포리스트 회귀: 모듈 참조'
titleSuffix: Azure Machine Learning
description: Azure 기계 학습에서 의사 결정 포리스트 회귀 모듈을 사용하여 의사 결정 트리의 집합을 기반으로 회귀 모델을 만드는 방법을 알아봅니다.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: 63d90a5239e6bf350d8a6b66f35157e4c7d15aee
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79456542"
---
# <a name="decision-forest-regression-module"></a>의사 결정 포리스트 회귀 모듈

이 문서에서는 Azure 기계 학습 디자이너(미리 보기)의 모듈에 대해 설명합니다.

이 모듈을 사용하여 의사 결정 트리의 앙상블을 기반으로 회귀 모델을 만듭니다.

모델을 구성한 후에는 레이블이 지정된 데이터 집합과 모델 [학습](./train-model.md) 모듈을 사용하여 모델을 학습해야 합니다. 그러면 학습된 모델을 예측에 사용할 수 있습니다. 

## <a name="how-it-works"></a>작동 방법

의사 결정 트리는 리프 노드(의사 결정)에 도달할 때까지 이진 트리 데이터 구조를 트래버스하며 각 인스턴스에 대해 일련의 간단한 테스트를 수행하는 비파라메트릭 모델입니다.

의사 결정 트리를 사용하는 경우의 이점은 다음과 같습니다.

- 학습 및 예측 중에 계산과 메모리 사용량 측면에서 모두 효율적입니다.

- 비선형 의사 결정 경계를 표시할 수 있습니다.

- 통합 기능 선택 및 분류를 수행하며 불필요한 데이터가 많은 기능이 있는 경우 복원이 가능합니다.

이 회귀 모델은 의사 결정 트리의 앙상블로 구성되어 있습니다. 회귀 결정 포리스트의 각 트리는 가우시안 분포를 예측으로 출력합니다. 집합은 나무의 앙상블을 통해 수행되어 모델의 모든 트리에 대한 결합 분포에 가장 가까운 가우시안 분포를 찾습니다.

이 알고리즘 및 구현에 대한 이론적 프레임워크에 대한 자세한 내용은 이 문서를 참조하십시오: [의사 결정 포리스트: 분류, 회귀, 밀도 추정, 매니폴드 학습 및 반 감독 학습을 위한 통합 프레임워크](https://www.microsoft.com/en-us/research/publication/decision-forests-a-unified-framework-for-classification-regression-density-estimation-manifold-learning-and-semi-supervised-learning/?from=http%3A%2F%2Fresearch.microsoft.com%2Fapps%2Fpubs%2Fdefault.aspx%3Fid%3D158806#)

## <a name="how-to-configure-decision-forest-regression-model"></a>의사 결정 포리스트 회귀 모델을 구성하는 방법

1. **파이프라인에 의사 결정 포리스트 회귀 모듈을 추가합니다.** **기계 학습,** **모델 초기화**및 회귀 에서 디자이너에서 모듈을 찾을 수 **있습니다.**

2. 모듈 속성을 열고 **Resampling 메서드의**경우 개별 트리를 만드는 데 사용되는 메서드를 선택합니다.  **Bagging** 또는 **복제에서**선택할 수 있습니다.

    - **배깅**: 배깅은 *부트 스트랩 집계라고도합니다.* 회귀 결정 포리스트의 각 트리는 예측을 통해 가우시안 분포를 출력합니다. 집계는 개별 트리에서 반환되는 모든 분포를 결합하여 주어진 가우시안 분포의 혼합의 순간과 일치하는 처음 두 순간이 있는 가우시안을 찾는 것입니다.

         자세한 내용은 [부트스트랩 집계에](https://wikipedia.org/wiki/Bootstrap_aggregating)대한 위키백과 항목을 참조하십시오.

    - **복제**: 복제에서 각 트리는 정확히 동일한 입력 데이터에 대해 학습됩니다. 분할 조건어가 각 트리 노드에 사용되는 결정은 임의로 유지되며 트리는 다양합니다.

         **복제** 옵션을 사용 하 여 교육 프로세스에 대 한 자세한 내용은 [컴퓨터 비전 및 의료 이미지 분석에 대 한 의사 결정 포리스트를 참조 하십시오. 크리미니시와 제이 샷턴. 스프링어 2013.](https://research.microsoft.com/projects/decisionforests/).

3. **트레이너 만들기 모드** 만들기 옵션을 설정하여 모델을 학습할 방법을 지정합니다.

    - **단일 매개 변수입니다.**

      모델을 어떻게 구성할지 아는 경우 특정 값 집합을 인수로 제공할 수 있습니다. 실험을 통해 이러한 값을 알았거나 지침으로 받았을 수 있습니다.

    - **매개 변수 범위**: 최상의 매개 변수를 잘 모르고 매개 변수 스윕을 실행하려는 경우 이 옵션을 선택합니다. 반복할 값 범위를 선택하고 [모델 하이퍼매개 변수 조정은](tune-model-hyperparameters.md) 제공한 설정의 가능한 모든 조합을 반복하여 최적의 결과를 생성하는 하이퍼매개 변수를 결정합니다. 



4. **의사 결정 트리 수의**경우 앙상블에서 작성할 의사 결정 트리의 총 수를 나타냅니다. 추가 의사 결정 트리를 만들면 적용 범위가 확대될 수 있지만 학습 시간이 증가됩니다.

    > [!TIP]
    > 이 값은 학습된 모델을 시각화할 때 표시되는 트리 수도 제어합니다. 단일 트리를 보거나 인쇄하려면 값을 1로 설정할 수 있습니다. 그러나 이는 하나의 트리(초기 매개 변수 집합이 있는 트리)만 생성되고 더 이상 반복이 수행되지 않는다는 것을 의미합니다.

5. **의사 결정 트리의 최대 깊이에**대 한 모든 의사 결정 트리의 최대 깊이 제한 하는 숫자를 입력 합니다. 트리의 수준을 늘리면 정밀도는 높아질 수 있지만 학습 시간이 더 길어지고 과잉 맞춤이 발생할 수 있습니다.

6. **노드당 임의 분할 수의**경우 트리의 각 노드를 빌드할 때 사용할 분할 수를 입력합니다. *분할은* 트리(노드)의 각 레벨에 있는 피처가 무작위로 분할된다는 것을 의미합니다.

7. **리프 노드당 최소 샘플 수의**경우 트리에서 터미널 노드(리프)를 만드는 데 필요한 최소 케이스 수를 나타냅니다.

     이 값을 늘려 새 규칙을 작성하기 위한 임계값을 늘립니다. 예를 들어, 기본값이 1이면 단일 사례만으로도 새 규칙을 하나 작성할 수 있습니다. 값을 5로 늘리면 학습 데이터에 동일한 조건을 만족하는 사례가 다섯 개 이상 있어야 합니다.


9. 레이블이 지정된 데이터 집합을 연결하고, 두 개 이하의 결과를 포함하는 단일 레이블 열을 선택하고, [모델 학습에](./train-model.md)연결합니다.

    - **트레이너 모드 만들기** 옵션을 **단일 매개 변수로**설정하면 [학습 모델](./train-model.md) 모듈을 사용하여 모델을 학습합니다.

   

10. 파이프라인을 제출합니다.

### <a name="results"></a>결과

교육이 완료된 후:

+ 학습된 모델의 스냅샷을 저장하려면 학습 모듈을 선택한 다음 오른쪽 패널의 출력 탭으로 **전환합니다.** 아이콘 레지스터 **모델을**클릭합니다.  저장된 모델을 모듈 트리에서 모듈로 찾을 수 있습니다. 

## <a name="next-steps"></a>다음 단계

Azure 기계 학습에 사용할 수 있는 [모듈 집합을](module-reference.md) 참조하십시오. 