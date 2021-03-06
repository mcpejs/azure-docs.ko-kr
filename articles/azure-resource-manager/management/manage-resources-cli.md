---
title: 리소스 관리 - Azure CLI
description: Azure CLI 및 Azure 리소스 관리자를 사용하여 리소스를 관리합니다. 리소스를 배포하고 삭제하는 방법을 보여 주면 됩니다.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: f00c1725201b0e49f80fec64e5d69b375ec7a233
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75485534"
---
# <a name="manage-azure-resources-by-using-azure-cli"></a>Azure CLI를 사용하여 Azure 리소스 관리

Azure [리소스 관리자와](overview.md) 함께 Azure CLI를 사용하여 Azure 리소스를 관리하는 방법에 대해 알아봅니다. 리소스 그룹을 관리하려는 경우 [Azure CLI를 사용하여 Azure 리소스 그룹 관리를](manage-resource-groups-cli.md)참조하십시오.

리소스 관리에 대한 기타 문서:

- [Azure 포털을 사용하여 Azure 리소스 관리](manage-resources-portal.md)
- [Azure PowerShell을 사용하여 Azure 리소스 관리](manage-resources-powershell.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>기존 리소스 그룹에 리소스 배포

Azure CLI를 사용하여 Azure 리소스를 직접 배포하거나 리소스 관리자 템플릿을 배포하여 Azure 리소스를 만들 수 있습니다.

### <a name="deploy-a-resource"></a>리소스 배포

다음 스크립트는 저장소 계정을 만듭니다.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account create --resource-group $resourceGroupName --name $storageAccountName --location $location --sku Standard_LRS --kind StorageV2 &&
az storage account show --resource-group $resourceGroupName --name $storageAccountName 
```

### <a name="deploy-a-template"></a>템플릿 배포

다음 스크립트는 저장소 계정을 만들기 위해 빠른 시작 템플릿을 배포합니다. 자세한 내용은 [빠른 시작: Visual Studio 코드를 사용하여 Azure 리소스 관리자 템플릿 만들기를](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell)참조하세요.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group deployment create --resource-group $resourceGroupName --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

자세한 내용은 [Resource Manager 템플릿과 Azure CLI로 리소스 배포](../templates/deploy-cli.md)를 참조하세요.

## <a name="deploy-a-resource-group-and-resources"></a>리소스 그룹 및 리소스 배포

리소스 그룹을 만들고 그룹에 리소스를 배포할 수 있습니다. 자세한 내용은 [리소스 그룹 만들기 및 리소스 배포](../templates/deploy-to-subscription.md#resource-group-and-resources)를 참조하세요.

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>여러 구독 또는 리소스 그룹에 리소스 배포

일반적으로 단일 리소스 그룹에 템플릿의 모든 리소스를 배포합니다. 그러나 일단의 리소스를 함께 배포하고 다른 리소스 그룹 또는 구독에 배치하려는 시나리오가 있습니다. 자세한 내용은 [여러 구독 또는 리소스 그룹에 Azure 리소스 배포를 참조합니다.](../templates/cross-resource-group-deployment.md)

## <a name="delete-resources"></a>리소스 삭제

다음 스크립트에서는 저장소 계정을 삭제하는 방법을 보여 주며, 이 스크립트는 저장소 계정을 삭제하는 방법을 보여 주며,

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --resource-group $resourceGroupName --name $storageAccountName 
```

Azure 리소스 관리자가 리소스 삭제를 명령하는 방법에 대한 자세한 내용은 [Azure 리소스 관리자 리소스 그룹 삭제를](delete-resource-group.md)참조하십시오.

## <a name="move-resources"></a>리소스 이동

다음 스크립트에서는 한 리소스 그룹에서 다른 리소스 그룹으로 저장소 계정을 제거하는 방법을 보여 주며 있습니다.

```azurecli-interactive
echo "Enter the source Resource Group name:" &&
read srcResourceGroupName &&
echo "Enter the destination Resource Group name:" &&
read destResourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
storageAccount=$(az resource show --resource-group $srcResourceGroupName --name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --query id --output tsv) &&
az resource move --destination-group $destResourceGroupName --ids $storageAccount
```

자세한 내용을 보려면 [새 리소스 그룹 또는 구독으로 리소스 이동](move-resource-group-and-subscription.md)을 참조하세요.

## <a name="lock-resources"></a>리소스 잠금

잠금을 사용하면 조직의 다른 사용자가 Azure 구독, 리소스 그룹 또는 리소스와 같은 중요한 리소스를 실수로 삭제하거나 수정할 수 없습니다. 

다음 스크립트는 저장소 계정을 잠그므로 계정을 삭제할 수 없습니다.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock create --name LockSite --lock-type CanNotDelete --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts 
```

다음 스크립트는 저장소 계정에 대한 모든 잠금을 가져옵니다.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock list --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --parent ""
```

다음 스크립트는 저장소 계정의 잠금을 삭제합니다.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
lockId=$(az lock show --name LockSite --resource-group $resourceGroupName --resource-type Microsoft.Storage/storageAccounts --resource-name $storageAccountName --output tsv --query id)&&
az lock delete --ids $lockId
```

자세한 내용은 [Azure 리소스 관리자를 사용 하 여 리소스 잠금](lock-resources.md)을 참조 합니다.

## <a name="tag-resources"></a>리소스 태그 지정

태그 지정은 리소스 그룹과 리소스를 논리적으로 구성하는 데 도움이 됩니다. 자세한 내용은 [태그 사용을 사용하여 Azure 리소스를 구성합니다.](tag-resources.md#azure-cli)

## <a name="manage-access-to-resources"></a>리소스에 대한 액세스 관리

[RBAC(역할 기반 액세스 제어)](../../role-based-access-control/overview.md)는 Azure에서 리소스에 대한 액세스를 관리하는 방법입니다. 자세한 내용은 [RBAC 및 Azure CLI를 사용하여 액세스 관리를](../../role-based-access-control/role-assignments-cli.md)참조하십시오.

## <a name="next-steps"></a>다음 단계

- Azure 리소스 관리자에 대해 알아보려면 [Azure 리소스 관리자 개요를](overview.md)참조하십시오.
- 리소스 관리자 템플릿 구문을 알아보려면 [Azure Resource Manager 템플릿의 구조 및 구문 이해를](../templates/template-syntax.md)참조하십시오.
- 템플릿을 개발하는 방법에 대한 자세한 내용은 [단계별 자습서를](/azure/azure-resource-manager/)참조하십시오.
- Azure 리소스 관리자 템플릿 스키마를 보려면 [템플릿 참조를](/azure/templates/)참조하십시오.
