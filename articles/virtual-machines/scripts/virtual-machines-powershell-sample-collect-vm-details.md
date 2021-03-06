---
title: PowerShell을 통해 구독의 모든 VM에 대한 세부 정보 수집
description: PowerShell을 통해 구독의 모든 VM에 대한 세부 정보 수집
services: virtual-machines-windows
documentationcenter: virtual-machines
author: v-miegge
manager: dcscontentpm
editor: v-miegge
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 07/01/2019
ms.author: v-miegge
ms.custom: mvc
ms.openlocfilehash: 237081380445f2b2e4168ee3afe9a3ed7544fc89
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74900195"
---
# <a name="collect-details-about-all-vms-in-a-subscription-with-powershell"></a>PowerShell을 통해 구독의 모든 VM에 대한 세부 정보 수집

이 스크립트는 제공된 구독에서 VM의 VM 이름, 리소스 그룹 이름, 지역, 가상 네트워크, 서브넷, 개인 IP 주소, OS 유형 및 공용 IP 주소를 포함하는 csv를 만듭니다.

[Azure 구독이](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing)없는 경우 시작하기 전에 [무료 계정을](https://azure.microsoft.com/free) 만듭니다.

## <a name="launch-azure-cloud-shell"></a>Azure Cloud Shell 시작

Azure Cloud Shell은 이 항목의 단계를 실행하는 데 무료로 사용할 수 있는 대화형 셸입니다. 공용 Azure 도구가 사전 설치되어 계정에서 사용하도록 구성되어 있습니다. 

Cloud Shell을 열려면 코드 블록의 오른쪽 위 모서리에 있는 **사용해 보세요**를 선택하기만 하면 됩니다. 로 이동하여 별도의 브라우저 탭에서 Cloud [https://shell.azure.com/powershell](https://shell.azure.com/powershell)Shell을 시작할 수도 있습니다. **복사를** 선택하여 코드 블록을 복사하고 클라우드 셸에 붙여넣은 다음 enter를 눌러 실행합니다.

## <a name="sample-script"></a>샘플 스크립트

```azurepowershell-interactive
#Provide the subscription Id where the VMs reside
$subscriptionId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

#Provide the name of the csv file to be exported
$reportName = "myReport.csv"

Select-AzSubscription $subscriptionId
$report = @()
$vms = Get-AzVM
$publicIps = Get-AzPublicIpAddress 
$nics = Get-AzNetworkInterface | ?{ $_.VirtualMachine -NE $null} 
foreach ($nic in $nics) { 
    $info = "" | Select VmName, ResourceGroupName, Region, VirturalNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress 
    $vm = $vms | ? -Property Id -eq $nic.VirtualMachine.id 
    foreach($publicIp in $publicIps) { 
        if($nic.IpConfigurations.id -eq $publicIp.ipconfiguration.Id) {
            $info.PublicIPAddress = $publicIp.ipaddress
            } 
        } 
        $info.OsType = $vm.StorageProfile.OsDisk.OsType 
        $info.VMName = $vm.Name 
        $info.ResourceGroupName = $vm.ResourceGroupName 
        $info.Region = $vm.Location 
        $info.VirturalNetwork = $nic.IpConfigurations.subnet.Id.Split("/")[-3] 
        $info.Subnet = $nic.IpConfigurations.subnet.Id.Split("/")[-1] 
        $info.PrivateIpAddress = $nic.IpConfigurations.PrivateIpAddress 
        $report+=$info 
    } 
$report | ft VmName, ResourceGroupName, Region, VirturalNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress 
$report | Export-CSV "$home/$reportName"
```

## <a name="script-explanation"></a>스크립트 설명
이 스크립트는 다음 명령을 사용하여 구독에서 VM의 세부 정보에 대한 csv 내보내기를 만듭니다. 테이블에 있는 각 명령은 명령에 해당하는 문서에 연결됩니다.

|명령|메모|
|-|-|
|[선택-아즈서브셀백서스](https://docs.microsoft.com/powershell/module/Az.Accounts/Set-AzContext)|현재 세션에서 사용할 cmdlets에 대한 테넌트, 구독 및 환경을 설정합니다.|
|[Get-AzVM](https://docs.microsoft.com/powershell/module/Az.Compute/Get-AzVM)|가상 머신의 속성을 가져옵니다.|
|[Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/Az.Network/Get-AzPublicIpAddress)|공용 IP 주소를 가져옵니다.|
|[겟-아즈네트워크인터페이스](https://docs.microsoft.com/powershell/module/Az.Network/Get-AzNetworkInterface)|네트워크 인터페이스를 가져옵니다.|

## <a name="next-steps"></a>다음 단계

Azure PowerShell 모듈에 대한 자세한 내용은 [Azure PowerShell 설명서](https://docs.microsoft.com/powershell/azure/overview)를 참조하세요.

추가 가상 머신 PowerShell 스크립트 샘플은 [Azure Windows VM 설명서](https://docs.microsoft.com/azure/virtual-machines/windows/powershell-samples?toc=/azure/virtual-machines/windows/toc.json)에서 확인할 수 있습니다.


