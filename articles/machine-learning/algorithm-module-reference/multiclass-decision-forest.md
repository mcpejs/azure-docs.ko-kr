---
title: '다중 클래스 의사 결정 포리스트: 모듈 참조'
titleSuffix: Azure Machine Learning
description: Azure 기계 학습에서 다중 클래스 의사 결정 포리스트 모듈을 사용하여 *의사 결정 포리스트* 알고리즘을 기반으로 기계 학습 모델을 만드는 방법을 알아봅니다.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 02/22/2020
ms.openlocfilehash: 9a0a70f94be337eedf8f8ba4cc17af896f7a03b0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79477547"
---
# <a name="multiclass-decision-forest-module"></a>다중 클래스 의사 결정 포리스트 모듈

이 문서에서는 Azure 기계 학습 디자이너(미리 보기)의 모듈에 대해 설명합니다.

이 모듈을 사용하여 *의사 결정 포리스트* 알고리즘을 기반으로 기계 학습 모델을 만듭니다. 의사 결정 포리스트는 태그가 지정된 데이터에서 학습하면서 일련의 의사 결정 트리를 빠르게 빌드하는 앙상블 모델입니다.

## <a name="more-about-decision-forests"></a>의사 결정 포리스트에 대한 자세한 내용은

의사 결정 포리스트 알고리즘은 분류를 위한 앙상블 학습 방법입니다. 알고리즘은 여러 의사 결정 트리를 작성한 다음 가장 인기 있는 출력 클래스에 *투표하여* 작동합니다. 투표는 분류 결정 포리스트의 각 트리가 레이블의 정규화되지 않은 주파수 히스토그램을 출력하는 집계의 한 형태입니다. 집계 프로세스는 이러한 히스토그램을 합산하고 결과를 정규화하여 각 레이블에 대한 "확률"을 얻습니다. 예측 신뢰도가 높은 나무는 앙상블의 최종 결정에 더 큰 비중을 두고 있습니다.

일반적으로 의사 결정 트리는 비파라메트릭 모델이므로 다양한 분포를 가진 데이터를 지원합니다. 각 트리에서는 각 클래스에 대해 간단한 테스트 시퀀스를 실행하여 리프 노드(의사 결정)에 도달할 때까지 트리 구조 수준을 늘립니다.

의사 결정 트리를 사용하는 경우 많은 이점이 제공됩니다.

+ 비선형 의사 결정 경계를 표시할 수 있습니다.
+ 학습 및 예측 중에 계산과 메모리 사용량 측면에서 효율적입니다.
+ 통합 기능 선택 및 분류를 수행합니다.
+ 잡음 기능이 있는 경우 탄력적입니다.

Azure 기계 학습의 의사 결정 포리스트 분류기는 의사 결정 트리의 집합으로 구성됩니다. 일반적으로 앙상블 모델은 단일 의사 결정 트리에 비해 적용 범위가 넓고 정확도가 높습니다. 자세한 내용은 [의사 결정 트리를](https://go.microsoft.com/fwlink/?LinkId=403677)참조하십시오.

## <a name="how-to-configure-multiclass-decision-forest"></a>다중 클래스 의사 결정 포리스트를 구성하는 방법

1. 디자이너의 파이프라인에 **다중 클래스 의사 결정 포리스트** 모듈을 추가합니다. 이 모듈은 **기계 학습,** **모델 초기화**및 **분류**에서 찾을 수 있습니다.

2. **속성을** 열려면 모듈을 두 번 클릭합니다.

3. **리샘플링 방법의**경우 개별 트리를 만드는 데 사용되는 방법을 선택합니다.  모음 만들기 또는 복제 중에서 선택할 수 있습니다.

    + **배깅**: 배깅은 *부트 스트랩 집계라고도합니다.* 이 메서드에서는 각 트리가 새 샘플에서 성장하며, 원본 의 크기인 데이터 집합이 있을 때까지 원래 데이터 집합을 임의로 샘플링하여 대체합니다. 모델의 출력은 집계의 한 형태인 *투표로*결합됩니다. 자세한 내용은 부트스트랩 집계에 대한 위키백과 항목을 참조하십시오.

    + **복제**: 복제에서 각 트리는 정확히 동일한 입력 데이터에 대해 학습됩니다. 분할 조건어가 각 트리 노드에 사용되는 지 의 결정은 임의로 유지되어 다양한 트리를 만듭니다.

   

4. **트레이너 만들기 모드** 만들기 옵션을 설정하여 모델을 학습할 방법을 지정합니다.

    + **단일 매개 변수**: 모델을 구성하는 방법을 알고 있는 경우 이 옵션을 선택하고 값 집합을 인수로 제공합니다.

    + **매개 변수 범위**: 최상의 매개 변수를 잘 모르고 매개 변수 스윕을 실행하려는 경우 이 옵션을 선택합니다. 반복할 값 범위를 선택하고 [모델 하이퍼매개 변수 조정은](tune-model-hyperparameters.md) 제공한 설정의 가능한 모든 조합을 반복하여 최적의 결과를 생성하는 하이퍼매개 변수를 결정합니다.   

5. **의사 결정 트리 수**: 앙상블에서 만들 수 있는 최대 의사 결정 트리 수를 입력합니다. 더 많은 의사 결정 트리를 만들면 잠재적으로 더 나은 커버리지를 얻을 수 있지만 교육 시간이 늘어칠 수 있습니다.

    이 값은 학습된 모델을 시각화할 때 결과에 표시되는 트리 수도 제어합니다. 단일 트리를 보거나 인쇄하려면 값을 1로 설정할 수 있습니다. 그러나 이는 하나의 트리만 생성할 수 있으며(초기 매개 변수 집합이 있는 트리) 추가 반복이 수행되지 않습니다.

6. **의사 결정 트리의 최대 깊이**: 모든 의사 결정 트리의 최대 깊이를 제한하기 위해 숫자를 입력합니다. 트리의 수준을 늘리면 정밀도는 높아질 수 있지만 학습 시간이 더 길어지고 과잉 맞춤이 발생할 수 있습니다.

7. **노드당 임의 분할 수**: 트리의 각 노드를 빌드할 때 사용할 분할 수를 입력합니다. *분할은* 트리(노드)의 각 레벨에 있는 피처가 무작위로 분할된다는 것을 의미합니다.

8. **리프 노드당 최소 샘플 수**: 트리에서 터미널 노드(리프)를 만드는 데 필요한 최소 케이스 수를 나타냅니다. 이 값을 늘려 새 규칙을 작성하기 위한 임계값을 늘립니다.

    예를 들어, 기본값이 1이면 단일 사례만으로도 새 규칙을 하나 작성할 수 있습니다. 값을 5로 늘리면 학습 데이터에 동일한 조건을 만족하는 사례가 다섯 개 이상 있어야 합니다.



10. 레이블이 지정된 데이터 집합과 교육 모듈 중 하나를 연결합니다.

    + **트레이너 만들기 모드를** 단일 매개 **변수로**설정하면 [학습 모델](./train-model.md) 모듈을 사용합니다.

11. 파이프라인을 제출합니다.



## <a name="next-steps"></a>다음 단계

Azure 기계 학습에 사용할 수 있는 [모듈 집합을](module-reference.md) 참조하십시오. 