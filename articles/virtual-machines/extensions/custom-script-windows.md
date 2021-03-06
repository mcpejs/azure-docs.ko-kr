---
title: Windows용 Azure 사용자 지정 스크립트 확장
description: 사용자 지정 스크립트 확장을 사용하여 Windows VM 구성 작업 자동화
services: virtual-machines-windows
manager: carmonm
author: bobbytreed
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/02/2019
ms.author: robreed
ms.openlocfilehash: 2c7cad2dfdcd55073a1cf09d79e5223b666ced5f
ms.sourcegitcommit: efefce53f1b75e5d90e27d3fd3719e146983a780
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/01/2020
ms.locfileid: "80478159"
---
# <a name="custom-script-extension-for-windows"></a>Windows용 사용자 지정 스크립트 확장

사용자 지정 스크립트 확장은 Azure 가상 머신에서 스크립트를 다운로드하고 실행합니다. 이 확장은 배포 후 구성, 소프트웨어 설치 또는 기타 구성 또는 관리 작업에 유용합니다. 스크립트를 Azure Storage 또는 GitHub에서 다운로드하거나 확장 런타임에서 Azure Portal에 제공할 수 있습니다. 사용자 지정 스크립트 확장은 Azure 리소스 관리자 템플릿과 통합되며 Azure CLI, PowerShell, Azure 포털 또는 Azure 가상 시스템 REST API를 사용하여 실행할 수 있습니다.

이 문서에서는 Azure PowerShell 모듈, Azure Resource Manager 템플릿을 사용하는 사용자 지정 스크립트 확장을 사용하는 방법과 Windows 시스템에서 문제 해결 단계를 자세히 설명하고 있습니다.

## <a name="prerequisites"></a>사전 요구 사항

> [!NOTE]  
> 해당 매개 변수와 동일한 VM을 사용하여 Update-AzVM을 실행하려는 경우에는 사용자 지정 스크립트 확장을 사용하지 마세요. 대기 시간이 길어집니다.  

### <a name="operating-system"></a>운영 체제

Windows용 사용자 지정 스크립트 확장은 확장 지원 되는 확장 OS에서 실행 됩니다., 자세한 내용은이 [Azure 확장 지원 운영 체제를](https://support.microsoft.com/help/4078134/azure-extension-supported-operating-systems)참조 하십시오.

### <a name="script-location"></a>스크립트 위치

Azure Blob 저장소 자격 증명을 사용하여 Azure Blob 저장소에 액세스하도록 확장을 구성할 수 있습니다. 스크립트 위치는 VM이 GitHub 또는 내부 파일 서버와 같은 해당 끝점으로 라우팅할 수 있는 한 어디에나 있을 수 있습니다.

### <a name="internet-connectivity"></a>인터넷 연결

GitHub 또는 Azure Storage와 같은 외부스크립트를 외부로 다운로드해야 하는 경우 추가 방화벽 및 네트워크 보안 그룹 포트를 열어야 합니다. 예를 들어 스크립트가 Azure 저장소에 있는 경우 [저장소에](../../virtual-network/security-overview.md#service-tags)대 한 Azure NSG 서비스 태그를 사용 하 여 액세스를 허용할 수 있습니다.

스크립트가 로컬 서버에 있는 경우에도 추가 방화벽이 필요할 수 있으며 네트워크 보안 그룹 포트를 열어야 합니다.

### <a name="tips-and-tricks"></a>팁과 요령

* 이 확장에 대 한 가장 높은 실패율 때문에 스크립트에 구문 오류, 오류 없이 실행 하는 스크립트를 테스트 하 고 또한 실패 한 위치를 쉽게 찾을 수 있도록 스크립트에 추가 로깅에 넣어.
* idempotent인 스크립트를 작성합니다. 이렇게 하면 실수로 다시 실행해도 시스템 변경이 발생하지 않습니다.
* 스크립트를 실행할 때 사용자 입력이 필요하지 않도록 합니다.
* 스크립트를 실행하는 데 허용되는 시간은 90분입니다. 더 오래 걸리면 확장을 프로비전하는 데 실패합니다.
* 스크립트 내에서 다시 부팅하지 마세요. 이 작업으로 인해 설치 중인 다른 확장에 문제가 발생합니다. 다시 부팅 후 확장은 다시 시작한 후 계속되지 않습니다.
* 재부팅을 유발하는 스크립트가 있는 경우 응용 프로그램을 설치하고 스크립트를 실행하면 Windows 예약 작업을 사용하여 재부팅을 예약하거나 DSC, Chef 또는 Puppet 확장과 같은 도구를 사용할 수 있습니다.
* 확장은 스크립트를 한 번만 실행합니다. 부팅할 때마다 스크립트를 실행하려면 확장을 사용하여 Windows 예약된 작업을 만들어야 합니다.
* 스크립트를 실행할 시기를 예약하려면 확장을 사용하여 Windows 예약된 작업을 만들어야 합니다.
* 스크립트를 실행하는 경우 Azure Portal 또는 CLI에서 ‘전환 중’ 확장 상태만 표시됩니다. 실행 중인 스크립트의 상태 업데이트를 더 자주 수행하려는 경우 사용자 고유의 솔루션을 만들어야 합니다.
* 사용자 지정 스크립트 확장은 프록시 서버를 기본적으로 지원하지 않지만 *Curl과* 같은 스크립트 내에서 프록시 서버를 지원하는 파일 전송 도구를 사용할 수 있습니다.
* 스크립트 또는 명령에서 사용할 수 있는 기본 디렉터리가 아닌 위치를 알고 있어야 하고, 이 상황을 처리할 논리가 있어야 합니다.
* 사용자 지정 스크립트 확장은 LocalSystem 계정에서 실행됩니다.

## <a name="extension-schema"></a>확장 스키마

사용자 지정 스크립트 확장 구성은 스크립트 위치 및 실행할 명령 등을 지정합니다. 이 구성을 구성 파일에 저장하거나, 명령줄에 지정하거나, Azure Resource Manager 템플릿에 지정할 수 있습니다.

중요한 데이터는 보호된 구성에 저장하면 암호화된 후 가상 머신 내에서만 해독됩니다. 보호된 구성은 실행 명령에 암호와 같은 기밀 정보가 포함될 때 유용합니다.

이러한 항목은 중요한 데이터로 처리하고 확장으로 보호되는 설정 구성에 지정되어야 합니다. Azure VM 확장으로 보호되는 설정 데이터는 암호화되어 대상 가상 머신에서만 해독됩니다.

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "virtualMachineName/config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
        "[variables('musicstoresqlName')]"
    ],
    "tags": {
        "displayName": "config-app"
    },
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.10",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "fileUris": [
                "script location"
            ],
            "timestamp":123456789
        },
        "protectedSettings": {
            "commandToExecute": "myExecutionCommand",
            "storageAccountName": "myStorageAccountName",
            "storageAccountKey": "myStorageAccountKey",
            "managedIdentity" : {}
        }
    }
}
```

> [!NOTE]
> managedId 속성은 저장소와 함께 사용할 수 **없습니다계정이름** 또는 storageAccountKey 속성

> [!NOTE]
> 한 번에 하나의 확장 버전만 VM에 설치할 수 있으며 동일한 VM에 대해 동일한 리소스 관리자 템플릿에서 사용자 지정 스크립트를 두 번 지정하면 실패합니다.

> [!NOTE]
> 이 스키마는 VirtualMachine 리소스 내에서 또는 독립 실행형 리소스로 사용할 수 있습니다. 이 확장이 ARM 템플릿에서 독립 실행형 리소스로 사용되는 경우 리소스 이름은 이 형식 "virtualMachineName/extensionName"이어야 합니다.

### <a name="property-values"></a>속성 값

| 속성 | 값/예제 | 데이터 형식 |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.Compute | 문자열 |
| type | CustomScriptExtension | 문자열 |
| typeHandlerVersion | 1.10 | int |
| fileUris(예) | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1 | array |
| timestamp(예) | 123456789 | 32비트 정수 |
| commandToExecute(예) | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 | 문자열 |
| storageAccountName(예) | examplestorageacct | 문자열 |
| storageAccountKey(예) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | 문자열 |
| 관리되는 정체성(예: | { } 또는 {"clientId": "31bb403aa-c3644-a7ff-d85fb6cd7232" } 또는 {objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" } | json 개체 |

>[!NOTE]
>이러한 속성 이름은 대/소문자를 구분합니다. 배포 문제를 방지하려면 다음과 같이 이름을 사용합니다.

#### <a name="property-value-details"></a>속성 값 세부 정보

* `commandToExecute`: (**필수**, 문자열) 실행할 진입점 스크립트입니다. 명령에 암호와 같은 비밀이 포함되어 있거나 fileUris가 중요한 경우 이 필드를 대신 사용합니다.
* `fileUris`: (옵션, 문자열 배열) 다운로드할 파일에 대한 URL입니다.
* `timestamp`(옵션, 32비트 정수)는 이 필드의 값을 변경하여 스크립트의 다시 실행을 트리거하는 데만 이 필드를 사용합니다.  모든 정수 값을 사용할 수 있습니다. 단, 이전 값과 달라야 합니다.
* `storageAccountName`: (옵션, 문자열) 스토리지 계정에 대한 이름입니다. 스토리지 자격 증명을 지정하는 경우 모든 `fileUris`는 Azure Blob에 대한 URL이어야 합니다.
* `storageAccountKey`: (선택 사항, 문자열) 스토리지 계정의 액세스 키
* `managedIdentity`: (선택 사항, json 개체) 파일을 다운로드하기위한 [관리 ID](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) (들)
  * `clientId`: 관리되는 ID의 클라이언트 ID(선택 사항, 문자열)
  * `objectId`: 관리되는 ID의 개체 ID(선택 사항, 문자열)

다음 값은 공용 또는 보호된 설정 중 하나에서 설정할 수 있습니다. 확장은 공용 및 보호된 설정 모두에 아래 값이 설정된 모든 구성을 거부합니다.

* `commandToExecute`

공용 설정을 사용하면 디버깅에 유용할 수 있지만 보호된 설정을 사용하는 것이 좋습니다.

공용 설정은 스크립트를 실행할 VM에 일반 텍스트로 보내집니다.  보호된 설정은 Azure 및 VM에만 알려진 키를 사용하여 암호화됩니다. 설정이 전송될 때 VM에 저장됩니다. 암호화된 값의 암호를 해독하는 데 사용되는 인증서는 VM에 저장되고, 필요한 경우 런타임 시 설정의 암호를 해독하는 데 사용됩니다.

####  <a name="property-managedidentity"></a>속성: 관리되는 정체성
> [!NOTE]
> 이 속성은 보호된 설정에서만 **지정해야 합니다.**

사용자 정의 스크립트 (버전 1.10 이후) "fileUris" 설정에서 제공 하는 URL에서 파일을 다운로드 하기 위한 [관리된 ID를](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) 지원 합니다. CustomScript를 사용하면 사용자가 SAS 토큰 또는 저장소 계정 키와 같은 비밀을 전달할 필요 없이 Azure Storage 개인 Blob 또는 컨테이너에 액세스할 수 있습니다.

이 기능을 사용하려면 사용자는 CustomScript가 실행될 것으로 예상되는 VM 또는 VMSS에 [시스템 할당](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet#add-a-system-assigned-identity) 또는 [사용자 할당ID를](https://docs.microsoft.com/azure/app-service/overview-managed-identity?tabs=dotnet#add-a-user-assigned-identity) 추가하고 [관리되는 ID 액세스를 Azure Storage 컨테이너 또는 Blob에 부여해야](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/tutorial-vm-windows-access-storage#grant-access)합니다.

대상 VM/VMSS에서 시스템 할당된 ID를 사용하려면 "관리ID" 필드를 빈 json 개체로 설정합니다. 

> 예제:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : {}
> }
> ```

대상 VM/VMSS에서 사용자 할당된 ID를 사용하려면 클라이언트 ID 또는 관리되는 ID의 개체 ID로 "관리 ID" 필드를 구성합니다.

> 예제:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "clientId": "31b403aa-c364-4240-a7ff-d85fb6cd7232" }
> }
> ```
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" }
> }
> ```

> [!NOTE]
> managedId 속성은 저장소와 함께 사용할 수 **없습니다계정이름** 또는 storageAccountKey 속성

## <a name="template-deployment"></a>템플릿 배포

Azure Resource Manager 템플릿을 사용하여 Azure VM 확장을 배포할 수 있습니다. 이전 섹션에서 자세히 설명한 JSON 스키마는 Azure 리소스 관리자 템플릿에서 배포 하는 동안 사용자 지정 스크립트 확장을 실행 하는 데 사용할 수 있습니다. 다음 샘플은 사용자 지정 스크립트 확장을 사용하는 방법을 보여 줍니다.

* [자습서: Azure Resource Manager 템플릿을 사용하여 가상 머신 확장 배포](../../azure-resource-manager/templates/template-tutorial-deploy-vm-extensions.md)
* [Windows 및 Azure SQL DB에 두 개의 계층 애플리케이션 배포](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows)

## <a name="powershell-deployment"></a>PowerShell 배포

`Set-AzVMCustomScriptExtension` 명령을 사용하여 사용자 지정 스크립트 확장을 기존 가상 머신에 추가할 수 있습니다. 자세한 내용은 [Set-AzVMCustomScriptExtension](/powershell/module/az.compute/set-azvmcustomscriptextension)을 참조하세요.

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName <resourceGroupName> `
    -VMName <vmName> `
    -Location myLocation `
    -FileUri <fileUrl> `
    -Run 'myScript.ps1' `
    -Name DemoScriptExtension
```

## <a name="additional-examples"></a>추가 예

### <a name="using-multiple-scripts"></a>여러 스크립트 사용

이 예제에서는 서버를 빌드하는 데 사용되는 세 개의 스크립트가 있습니다. **commandToExecute** 는 첫 번째 스크립트를 호출한 다음 다른 스크립트가 호출되는 방법에 대한 옵션이 있습니다. 예를 들어 올바른 오류 처리, 로깅 및 상태 관리를 통해 실행을 제어하는 마스터 스크립트를 가질 수 있습니다. 스크립트는 실행을 위해 로컬 컴퓨터에 다운로드됩니다. 예를 들어 `1_Add_Tools.ps1` 스크립트에 `2_Add_Features.ps1` 추가하여 `.\2_Add_Features.ps1` 호출하고 에서 정의한 다른 스크립트에 `$settings`대해 이 프로세스를 반복합니다.

```powershell
$fileUri = @("https://xxxxxxx.blob.core.windows.net/buildServer1/1_Add_Tools.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/2_Add_Features.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/3_CompleteInstall.ps1")

$settings = @{"fileUris" = $fileUri};

$storageAcctName = "xxxxxxx"
$storageKey = "1234ABCD"
$protectedSettings = @{"storageAccountName" = $storageAcctName; "storageAccountKey" = $storageKey; "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File 1_Add_Tools.ps1"};

#run command
Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "buildserver1" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -Settings $settings    `
    -ProtectedSettings $protectedSettings `
```

### <a name="running-scripts-from-a-local-share"></a>로컬 공유에서 스크립트 실행

이 예제에서는 스크립트 위치에 로컬 SMB 서버를 사용할 수 있습니다. 이렇게 하면 **명령토실행**을 제외한 다른 설정을 제공할 필요가 없습니다.

```powershell
$protectedSettings = @{"commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File \\filesvr\build\serverUpdate1.ps1"};

Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "serverUpdate"
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -ProtectedSettings $protectedSettings

```

### <a name="how-to-run-custom-script-more-than-once-with-cli"></a>CLI를 사용하여 두 번 이상 사용자 지정 스크립트를 실행하는 방법

사용자 지정 스크립트 확장을 두 번 이상 실행하려는 경우 다음 조건에서만 이 작업을 수행할 수 있습니다.

* 확장 **이름** 매개 변수는 확장의 이전 배포와 동일합니다.
* 그렇지 않으면 구성을 업데이트하여 명령이 다시 실행되지 않습니다. 타임스탬프와 같은 명령으로 동적 속성을 추가할 수 있습니다.

또는 [ForceUpdateTag](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension.forceupdatetag) 속성을 **true로**설정할 수 있습니다.

### <a name="using-invoke-webrequest"></a>호출-웹 요청 사용

스크립트에서 [Invoke-WebRequest를](/powershell/module/microsoft.powershell.utility/invoke-webrequest) 사용하는 경우 매개 변수를 `-UseBasicParsing` 지정해야 하며 자세한 상태를 확인할 때 다음과 같은 오류가 발생합니다.

```error
The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
```
## <a name="virtual-machine-scale-sets"></a>Virtual Machine Scale Sets

축척 집합에 사용자 지정 스크립트 확장을 배포하려면 [AzVmss 확장 추가를](https://docs.microsoft.com/powershell/module/az.compute/add-azvmssextension?view=azps-3.3.0) 참조하십시오.

## <a name="classic-vms"></a>클래식 VM

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

클래식 VM에 사용자 지정 스크립트 확장을 배포하려면 Azure 포털 또는 클래식 Azure PowerShell cmdlet을 사용할 수 있습니다.

### <a name="azure-portal"></a>Azure portal

클래식 VM 리소스로 이동합니다. **설정**에서 **확장을 선택합니다.**

**+ 추가를** 클릭하고 리소스 목록에서 **사용자 지정 스크립트 확장을**선택합니다.

확장 **설치** 페이지에서 로컬 PowerShell 파일을 선택하고 인수를 입력하고 **확인을**클릭합니다.

### <a name="powershell"></a>PowerShell

기존 가상 컴퓨터에 사용자 지정 스크립트 확장을 추가하는 데 사용할 수 있는 [Set-AzureVMCustomScriptExtension](/powershell/module/servicemanagement/azure/set-azurevmcustomscriptextension) cmdlet을 사용합니다.

```powershell
# define your file URI
$fileUri = 'https://xxxxxxx.blob.core.windows.net/scripts/Create-File.ps1'

# create vm object
$vm = Get-AzureVM -Name <vmName> -ServiceName <cloudServiceName>

# set extension
Set-AzureVMCustomScriptExtension -VM $vm -FileUri $fileUri -Run 'Create-File.ps1'

# update vm
$vm | Update-AzureVM
```

## <a name="troubleshoot-and-support"></a>문제 해결 및 지원

### <a name="troubleshoot"></a>문제 해결

확장 배포 상태에 대한 데이터는 Azure PowerShell 모듈을 사용하여 Azure Portal에서 검색할 수 있습니다. 지정된 VM에 대한 확장의 배포 상태를 보려면 다음 명령을 실행합니다.

```powershell
Get-AzVMExtension -ResourceGroupName <resourceGroupName> -VMName <vmName> -Name myExtensionName
```

확장자 출력은 대상 가상 머신의 다음 폴더 아래에 있는 파일에 기록됩니다.

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

지정된 파일은 대상 가상 머신의 다음 폴더에 다운로드됩니다.

```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>
```

여기서 `<n>`은 확장의 실행 간에 변경될 수 있는 10진수 정수입니다.  `1.*` 값은 확장의 현재 실제 `typeHandlerVersion` 값과 일치합니다.  예를 들어, 실제 디렉터리는 `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2`일 수 있습니다.  

`commandToExecute` 명령을 실행하는 경우 확장에서 이 디렉터리(예: `...\Downloads\2`)를 현재 작업 디렉터리로 설정합니다. 이 프로세스로 `fileURIs` 속성을 통해 다운로드된 파일을 배치하는 상대 경로를 사용할 수 있습니다. 예제는 아래 테이블을 참조하세요.

시간이 지남에 따라 절대 다운로드 경로가 달라질 수 있으므로 가능한 경우 `commandToExecute` 문자열에서 상대 스크립트/파일 경로를 옵트인하는 것이 좋습니다. 다음은 그 예입니다.

```json
"commandToExecute": "powershell.exe . . . -File \"./scripts/myscript.ps1\""
```

속성 목록을 통해 다운로드한 파일에 대해 첫 `fileUris` 번째 URI 세그먼트 이후의 경로 정보가 유지됩니다.  아래 테이블에 표시된 대로 다운로드된 파일은 다운로드 하위 디렉터리에 매핑되어 `fileUris` 값의 구조를 반영합니다.  

#### <a name="examples-of-downloaded-files"></a>다운로드된 파일의 예

| fileUris의 URI | 다운로드된 상대 위치 | 절대 다운로드 위치 <sup>1</sup> |
| ---- | ------- |:--- |
| `https://someAcct.blob.core.windows.net/aContainer/scripts/myscript.ps1` | `./scripts/myscript.ps1` |`C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\scripts\myscript.ps1`  |
| `https://someAcct.blob.core.windows.net/aContainer/topLevel.ps1` | `./topLevel.ps1` | `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\topLevel.ps1` |

<sup>1</sup> 절대 디렉터리 경로는 VM의 수명 동안 변경되지만 CustomScript 확장을 한 번실행하지는 않습니다.

### <a name="support"></a>고객 지원팀

이 문서의 어느 시점에서든 도움이 필요한 경우 [MSDN Azure 및 스택 오버플로 포럼의](https://azure.microsoft.com/support/forums/)Azure 전문가에게 문의할 수 있습니다. 또는 Azure 지원 인시던트를 제출할 수도 있습니다. [Azure 지원 사이트로](https://azure.microsoft.com/support/options/) 이동하여 지원 받기를 선택합니다. Azure 지원을 사용하는 방법에 대한 자세한 내용은 [Microsoft Azure 지원 FAQ](https://azure.microsoft.com/support/faq/)를 참조하세요.
