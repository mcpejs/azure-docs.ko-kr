---
title: 자습서 - 매개 변수 파일을 사용하여 템플릿 배포
description: Azure Resource Manager 템플릿을 배포하는 데 사용할 값이 포함된 매개 변수 파일을 사용합니다.
author: mumian
ms.date: 10/04/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 6a12d92c0cfb9d86ebf4c335c351944997f79b4e
ms.sourcegitcommit: 984c5b53851be35c7c3148dcd4dfd2a93cebe49f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2020
ms.locfileid: "76773159"
---
# <a name="tutorial-use-parameter-files-to-deploy-your-resource-manager-template"></a>자습서: 매개 변수 파일을 사용하여 Resource Manager 템플릿 배포

이 자습서에서는 [매개 변수 파일](parameter-files.md)을 사용하여 배포 중에 전달하는 값을 저장하는 방법을 알아봅니다. 이전 자습서에서는 배포 명령에 인라인 매개 변수를 사용했습니다. 이 방식은 템플릿을 테스트하는 데 적당하지만 배포를 자동화하는 경우에는 환경에 맞는 값 세트를 전달하는 것이 더 쉬울 수 있습니다. 매개 변수 파일을 사용하면 특정 환경에 대한 매개 변수 값을 쉽게 패키징할 수 있습니다. 이 자습서에서는 개발 및 프로덕션 환경에 대한 매개 변수 파일을 생성합니다. 완료하는 데 **12분** 정도 걸립니다.

## <a name="prerequisites"></a>사전 요구 사항

[태그에 대한 자습서](template-tutorial-add-tags.md)를 완료하는 것이 좋지만 필수는 아닙니다.

Resource Manager Tools 확장이 포함된 Visual Studio Code 및 Azure PowerShell 또는 Azure CLI가 있어야 합니다. 자세한 내용은 [템플릿 도구](template-tutorial-create-first-template.md#get-tools)를 참조하세요.

## <a name="review-template"></a>템플릿 검토

템플릿에는 배포 중에 제공할 수 있는 매개 변수가 많이 있습니다. 이전 자습서의 끝 부분에서, 템플릿은 다음과 같았습니다.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.json":::

이 템플릿은 잘 작동하지만 이제 템플릿에 전달하는 매개 변수를 쉽게 관리하려고 합니다.

## <a name="add-parameter-files"></a>매개 변수 파일 추가

매개 변수 파일은 템플릿과 유사한 구조의 JSON 파일입니다. 배포 중에 전달할 매개 변수 값을 이 파일에 제공합니다.

VS Code에서 다음 내용이 포함된 새 파일을 만듭니다. 이 파일을 **azuredeploy.parameters.dev.json**이라는 이름으로 저장합니다.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.dev.json":::

이 파일은 개발 환경에 대한 매개 변수 파일입니다. 스토리지 계정에 Standard_LRS를 사용하고, 리소스 이름에 **dev**를 접두사로 사용하고 **Environment** 태그를 **Dev**로 설정합니다.

다시, 다음 내용이 포함된 새 파일을 만듭니다. 이 파일을 **azuredeploy.parameters.prod.json**이라는 이름으로 저장합니다.

:::code language="json" source="~/resourcemanager-templates/get-started-with-templates/add-tags/azuredeploy.parameters.prod.json":::

이 파일은 프로덕션 환경에 대한 매개 변수 파일입니다. 스토리지 계정에 Standard_GRS를 사용하고, 리소스 이름에 **contoso**를 접두사로 사용하고 **Environment** 태그를 **Production**으로 설정합니다. 실제 프로덕션 환경에서는 무료가 아닌 SKU로 App Service를 사용할 수 있지만 이 자습서에서는 이 SKU를 계속 사용합니다.

## <a name="deploy-template"></a>템플릿 배포

Azure CLI 또는 Azure PowerShell을 사용하여 템플릿을 배포합니다.

템플릿의 최종 테스트로, 두 가지 리소스 그룹을 새로 만들어 보겠습니다. 하나는 개발 환경용이고 다른 하나는 프로덕션 환경용입니다.

먼저 개발 환경에 배포합니다.

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$templateFile = "{provide-the-path-to-the-template-file}"
$parameterFile="{path-to-azuredeploy.parameters.dev.json}"
New-AzResourceGroup `
  -Name myResourceGroupDev `
  -Location "East US"
New-AzResourceGroupDeployment `
  -Name devenvironment `
  -ResourceGroupName myResourceGroupDev `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
templateFile="{provide-the-path-to-the-template-file}"
az group create \
  --name myResourceGroupDev \
  --location "East US"
az group deployment create \
  --name devenvironment \
  --resource-group myResourceGroupDev \
  --template-file $templateFile \
  --parameters azuredeploy.parameters.dev.json
```

---

이제 프로덕션 환경에 배포합니다.

# <a name="powershelltabazure-powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$parameterFile="{path-to-azuredeploy.parameters.prod.json}"
New-AzResourceGroup `
  -Name myResourceGroupProd `
  -Location "West US"
New-AzResourceGroupDeployment `
  -Name prodenvironment `
  -ResourceGroupName myResourceGroupProd `
  -TemplateFile $templateFile `
  -TemplateParameterFile $parameterFile
```

# <a name="azure-clitabazure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group create \
  --name myResourceGroupProd \
  --location "West US"
az group deployment create \
  --name prodenvironment \
  --resource-group myResourceGroupProd \
  --template-file $templateFile \
  --parameters azuredeploy.parameters.prod.json
```

---

## <a name="verify-deployment"></a>배포 확인

Azure Portal에서 리소스 그룹을 탐색하여 배포를 확인할 수 있습니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
1. 왼쪽 메뉴에서 **리소스 그룹**을 선택합니다.
1. 이 자습서에서 배포한 두 가지 새 리소스 그룹이 보입니다.
1. 리소스 그룹을 선택하고 배포된 리소스를 살펴봅니다. 해당 환경에 대한 매개 변수 파일에 지정한 값과 일치하는 것을 확인합니다.

## <a name="clean-up-resources"></a>리소스 정리

1. Azure Portal의 왼쪽 메뉴에서 **리소스 그룹**을 선택합니다.
2. **이름으로 필터링** 필드에서 리소스 그룹 이름을 입력합니다. 이 시리즈를 완료했으면, 삭제할 리소스 그룹이 세 개(myResourceGroup, myResourceGroupDev 및 myResourceGroupProd) 있습니다.
3. 해당 리소스 그룹 이름을 선택합니다.
4. 위쪽 메뉴에서 **리소스 그룹 삭제**를 선택합니다.

## <a name="next-steps"></a>다음 단계

축하합니다, Azure에 템플릿을 배포하는 방법을 마쳤습니다. 의견이나 제안 사항이 있으면 사용자 의견 섹션에서 알려주십시오. 감사합니다.

템플릿에 대한 고급 개념을 자세히 살펴볼 준비가 되었습니다. 다음 자습서에서는 배포할 리소스를 정의하는 데 유용한 템플릿 참조 설명서를 사용하는 방법에 대해 자세히 설명합니다.

> [!div class="nextstepaction"]
> [템플릿 참조 활용](template-tutorial-create-encrypted-storage-accounts.md)
