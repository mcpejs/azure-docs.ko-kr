---
title: 자습서 - Terraform 및 HCL로 Azure VM 클러스터 만들기
description: 이 자습서에서는 Terraform 및 HCL을 사용하여 Azure에서 부하 분산 장치가 있는 Linux 가상 머신 클러스터를 만듭니다.
keywords: azure devops terraform vm 가상 머신 클러스터
ms.topic: tutorial
ms.date: 03/09/2020
ms.openlocfilehash: ae1b8eac15309ff27297d9472e70d32e68acaaac
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/24/2020
ms.locfileid: "78945265"
---
# <a name="tutorial-create-an-azure-vm-cluster-with-terraform-and-hcl"></a>자습서: Terraform 및 HCL로 Azure VM 클러스터 만들기

이 자습서에서는 [HCL](https://www.terraform.io/docs/configuration/syntax.html)을 사용하여 작은 컴퓨팅 클러스터를 만드는 방법을 보여 줍니다. 

다음 작업을 수행하는 방법을 알아봅니다.

> [!div class="checklist"]
> * Azure 인증을 설정합니다.
> * Terraform 구성 파일을 만듭니다.
> * Terraform 구성 파일을 사용하여 부하 분산 장치를 만듭니다.
> * Terraform 구성 파일을 사용하여 가용성 세트에 두 개의 Linux VM을 배포합니다.
> * Terraform을 초기화합니다.
> * Terraform 실행 계획을 만듭니다.
> * Terraform 실행 계획을 적용하여 Azure 리소스를 만듭니다.

## <a name="1-set-up-azure-authentication"></a>1. 인증 설정

> [!NOTE]
> [Terraform 환경 변수를 사용](terraform-install-configure.md)하거나 [Azure Cloud Shell](terraform-cloud-shell.md)에서 자습서를 실행하는 경우 이 섹션을 건너뜁니다.

이 섹션에서는 Azure 서비스 보안 주체와, 보안 주체의 자격 증명을 포함하는 Terraform 구성 파일을 두 개를 생성합니다.

1. Terraform에서 Azure로 리소스를 프로비전하도록 [Azure AD 서비스 사용자를 설정합니다](terraform-install-configure.md#set-up-terraform-access-to-azure). 보안 주체를 만드는 동안 구독 ID, 테넌트, 앱 ID, 암호의 값을 기록해 둡니다.

2. 명령 프롬프트를 엽니다.

3. Terraform 파일을 저장하는 빈 디렉터리를 만듭니다.

4. 변수 선언이 담긴 새 파일을 만듭니다. `.tf` 확장자를 사용하여 원하는 대로 이 파일의 이름을 지정하면 됩니다.

5. 변수 선언 파일에 다음 코드를 복사합니다.

   ```hcl
   variable subscription_id {}
   variable tenant_id {}
   variable client_id {}
   variable client_secret {}
  
   provider "azurerm" {
      version = "~>1.40"
     
      subscription_id = var.subscription_id
      tenant_id = var.tenant_id
      client_id = var.client_id
      client_secret = var.client_secret
   }
   ```

6. Terraform 변수에 대한 값이 담긴 새 파일을 만듭니다. Terraform이 이름이 `terraform.tfvars`인(또는 `*.auto.tfvars` 패턴을 따르는) 파일이 현재 디렉터리에 있으면 자동으로 로드하기 때문에 일반적으로 Terraform 변수 파일의 이름은 `terraform.tfvars`로 명명됩니다. 

7. 변수 파일에 다음 코드를 복사합니다. 다음과 같이 자리 표시자를 바꿉니다. `subscription_id`에는 `az account set`을 실행할 때 지정한 Azure 구독 ID를 사용합니다. `tenant_id`에는 `az ad sp create-for-rbac`에서 반환된 `tenant` 값을 사용합니다. `client_id`에는 `az ad sp create-for-rbac`에서 반환된 `appId` 값을 사용합니다. `client_secret`에는 `az ad sp create-for-rbac`에서 반환된 `password` 값을 사용합니다.

   ```hcl
   subscription_id = "<azure-subscription-id>"
   tenant_id = "<tenant-returned-from-creating-a-service-principal>"
   client_id = "<appId-returned-from-creating-a-service-principal>"
   client_secret = "<password-returned-from-creating-a-service-principal>"
   ```

## <a name="2-create-a-terraform-configuration-file"></a>2. Terraform 구성 파일 만들기

이 섹션에서는 인프라에 대한 리소스 정의가 포함된 파일을 만듭니다.

1. 이름이 `main.tf`인 새 파일을 만듭니다. 

2. 다음 샘플 리소스 정의를 새로 만든 `main.tf` 파일에 복사합니다. 

   ```hcl
   resource "azurerm_resource_group" "test" {
    name     = "acctestrg"
    location = "West US 2"
   }

   resource "azurerm_virtual_network" "test" {
    name                = "acctvn"
    address_space       = ["10.0.0.0/16"]
    location            = azurerm_resource_group.test.location
    resource_group_name = azurerm_resource_group.test.name
   }

   resource "azurerm_subnet" "test" {
    name                 = "acctsub"
    resource_group_name  = azurerm_resource_group.test.name
    virtual_network_name = azurerm_virtual_network.test.name
    address_prefix       = "10.0.2.0/24"
   }

   resource "azurerm_public_ip" "test" {
    name                         = "publicIPForLB"
    location                     = azurerm_resource_group.test.location
    resource_group_name          = azurerm_resource_group.test.name
    allocation_method            = "Static"
   }

   resource "azurerm_lb" "test" {
    name                = "loadBalancer"
    location            = azurerm_resource_group.test.location
    resource_group_name = azurerm_resource_group.test.name

    frontend_ip_configuration {
      name                 = "publicIPAddress"
      public_ip_address_id = azurerm_public_ip.test.id
    }
   }

   resource "azurerm_lb_backend_address_pool" "test" {
    resource_group_name = azurerm_resource_group.test.name
    loadbalancer_id     = azurerm_lb.test.id
    name                = "BackEndAddressPool"
   }

   resource "azurerm_network_interface" "test" {
    count               = 2
    name                = "acctni${count.index}"
    location            = azurerm_resource_group.test.location
    resource_group_name = azurerm_resource_group.test.name

    ip_configuration {
      name                          = "testConfiguration"
      subnet_id                     = azurerm_subnet.test.id
      private_ip_address_allocation = "dynamic"
    }
   }

   resource "azurerm_managed_disk" "test" {
    count                = 2
    name                 = "datadisk_existing_${count.index}"
    location             = azurerm_resource_group.test.location
    resource_group_name  = azurerm_resource_group.test.name
    storage_account_type = "Standard_LRS"
    create_option        = "Empty"
    disk_size_gb         = "1023"
   }

   resource "azurerm_availability_set" "avset" {
    name                         = "avset"
    location                     = azurerm_resource_group.test.location
    resource_group_name          = azurerm_resource_group.test.name
    platform_fault_domain_count  = 2
    platform_update_domain_count = 2
    managed                      = true
   }

   resource "azurerm_virtual_machine" "test" {
    count                 = 2
    name                  = "acctvm${count.index}"
    location              = azurerm_resource_group.test.location
    availability_set_id   = azurerm_availability_set.avset.id
    resource_group_name   = azurerm_resource_group.test.name
    network_interface_ids = [element(azurerm_network_interface.test.*.id, count.index)]
    vm_size               = "Standard_DS1_v2"

    # Uncomment this line to delete the OS disk automatically when deleting the VM
    # delete_os_disk_on_termination = true

    # Uncomment this line to delete the data disks automatically when deleting the VM
    # delete_data_disks_on_termination = true

    storage_image_reference {
      publisher = "Canonical"
      offer     = "UbuntuServer"
      sku       = "16.04-LTS"
      version   = "latest"
    }

    storage_os_disk {
      name              = "myosdisk${count.index}"
      caching           = "ReadWrite"
      create_option     = "FromImage"
      managed_disk_type = "Standard_LRS"
    }

    # Optional data disks
    storage_data_disk {
      name              = "datadisk_new_${count.index}"
      managed_disk_type = "Standard_LRS"
      create_option     = "Empty"
      lun               = 0
      disk_size_gb      = "1023"
    }

    storage_data_disk {
      name            = element(azurerm_managed_disk.test.*.name, count.index)
      managed_disk_id = element(azurerm_managed_disk.test.*.id, count.index)
      create_option   = "Attach"
      lun             = 1
      disk_size_gb    = element(azurerm_managed_disk.test.*.disk_size_gb, count.index)
    }

    os_profile {
      computer_name  = "hostname"
      admin_username = "testadmin"
      admin_password = "Password1234!"
    }

    os_profile_linux_config {
      disable_password_authentication = false
    }

    tags = {
      environment = "staging"
    }
   }
   ```

## <a name="3-initialize-terraform"></a>3. Terraform 초기화 

[terraform init 명령](https://www.terraform.io/docs/commands/init.html)은 이전 섹션에서 만든 Terraform 구성 파일이 담긴 디렉터리를 초기화하는 데 사용됩니다. 새 Terraform 구성 파일을 작성한 후에는 항상 `terraform init` 명령을 실행하는 것이 좋습니다. 

> [!TIP]
> `terraform init` 명령은 멱등으로, 동일한 결과 생성하는 동안 반복적으로 호출될 수 있습니다. 따라서 공동 작업 환경에 있으며 구성 파일의 변경 가능성이 있는 경우 계획을 실행하거나 적용하기 전에 항상 `terraform init` 명령을 호출하는 것이 좋습니다.

Terraform을 초기화하려면 다음 명령을 실행합니다.

  ```bash
  terraform init
  ```

  ![Terraform 초기화](media/terraform-create-vm-cluster-with-infrastructure/terraform-init.png)

## <a name="4-create-a-terraform-execution-plan"></a>4. Terraform 실행 계획 만들기

[Terraform Plan 명령](https://www.terraform.io/docs/commands/plan.html)은 실행 계획을 만드는 데 사용됩니다. 실행 계획을 만들기 위해 Terraform이 현재 디렉터리에 있는 모든 `.tf` 파일을 집계합니다. 

[out 매개 변수](https://www.terraform.io/docs/commands/plan.html#out-path)는 실행 계획을 출력 파일에 저장합니다. 이 기능은 다중 개발 환경에서 일반적인 동시성 문제를 해결합니다. 출력 파일에서 해결하는 이러한 문제 중 하나는 다음과 같습니다.

1. Dev 1은 구성 파일을 만듭니다.
1. Dev 2는 구성 파일을 수정합니다.
1. Dev 1은 구성 파일을 적용(실행)합니다.
1. Dev 1은 Dev 2가 구성을 수정했다는 것을 알지 못하는 예기치 않은 결과를 가져옵니다.

출력 파일을 지정하는 Dev 1은 Dev 2가 Dev 1에 영향을 주지 않습니다. 

실행 계획을 저장할 필요가 없으면 다음 명령을 실행합니다.

  ```bash
  terraform plan
  ```

실행 계획을 저장해야 하는 경우 다음 명령을 실행합니다. 자리 표시자를 사용자 환경에 적합한 값으로 바꿉니다.

  ```bash
  terraform plan -out=<path>
  ```

또 다른 유용한 매개 변수는 [-var 파일](https://www.terraform.io/docs/commands/plan.html#var-file-foo)입니다.

기본적으로 Terraform은 다음과 같이 변수 파일을 찾으려고 시도합니다.
- 파일 이름 `terraform.tfvars`
- 다음 패턴을 사용하여 이름이 지정된 파일입니다. `*.auto.tfvars`

그러나 변수 파일은 위의 두 가지 규칙 중 하나를 따를 필요가 없습니다. 이 경우 변수 파일 이름에 확장명이 포함되지 않는 `-var-file` 매개 변수를 사용하여 변수 파일 이름을 지정합니다. 다음 예제에서는 이 점에 대해 설명합니다.

```hcl
terraform plan -var-file <my-variables-file>
```

Terraform은 구성 파일에 지정된 상태를 구현하는 데 필요한 작업을 결정합니다.

![Terraform 실행 계획 만들기](media/terraform-create-vm-cluster-with-infrastructure/terraform-plan.png)

## <a name="5-apply-the-terraform-execution-plan"></a>5. Terraform 실행 계획 적용

이 자습서의 마지막 단계는[terraform apply 명령](https://www.terraform.io/docs/commands/apply.html)을 사용하여 `terraform plan` 명령에서 생성한 동작 집합을 적용하는 것입니다.

최신 실행 계획만 적용하려면 다음 명령을 실행합니다.

  ```bash
  terraform apply
  ```

이전에 저장한 실행 계획만 적용하려면 다음 명령을 실행합니다. 자리 표시자를 사용자 환경에 적합한 값으로 바꿉니다.

  ```bash
  terraform apply <path>
  ```

![Terraform 실행 계획 적용](media/terraform-create-vm-cluster-with-infrastructure/terraform-apply.png)

## <a name="next-steps"></a>다음 단계

> [!div class="nextstepaction"] 
> [Terraform을 사용하여 Azure 가상 머신 확장 집합 만들기](terraform-create-vm-scaleset-network-disks-hcl.md)