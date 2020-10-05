---
title: 创建 NAT 网关 - 资源管理器模板
titleSuffix: Azure Virtual Network NAT
description: 本快速入门介绍如何使用 Azure 资源管理器模板创建 NAT 网关。
services: load-balancer
documentationcenter: na
manager: digimobile
Customer intent: I want to create a NAT gateway by using an Azure Resource Manager template so that I can provide outbound connectivity for my virtual machines.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 03/09/2020
author: rockboyfor
ms.date: 10/05/2020
ms.testscope: yes
ms.testdate: 07/13/2020
ms.author: v-yeche
ms.custom: subject-armqs
ms.openlocfilehash: 7a3faaacd73563778dda66becfdeaba43ec5b482
ms.sourcegitcommit: 29a49e95f72f97790431104e837b114912c318b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "91564175"
---
<!--Verified successfully-->
# <a name="create-a-nat-gateway---resource-manager-template"></a>创建 NAT 网关 - 资源管理器模板

通过 Azure 资源管理器模板完成虚拟网络 NAT 入门。  此模板部署虚拟网络、NAT 网关资源和 Ubuntu 虚拟机。 Ubuntu 虚拟机将部署到与 NAT 网关资源关联的子网。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="create-a-nat-gateway-and-supporting-resources"></a>创建 NAT 网关和支持资源

此模板配置为创建 

* 虚拟网络 
* NAT 网关资源
* Ubuntu 虚拟机

Ubuntu VM 部署到与 NAT 网关资源关联的子网。

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速入门模板](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmname": {
            "defaultValue": "myVM",
            "type": "String",
            "metadata": {
              "description": "Name of the virtual machine"
            }
        },
         "vmsize": {
            "defaultValue": "Standard_D2s_v3",
            "type": "String",
            "metadata": {
              "description": "Size of the virtual machine"
            }
        },
        "vnetname": {
            "defaultValue": "myVnet",
            "type": "String",
            "metadata": {
              "description": "Name of the virtual network"
            }
        },
        "subnetname": {
            "defaultValue": "mySubnet",
            "type": "String",
            "metadata": {
                "description": "Name of the subnet for virtual network"
            }
        },
        "vnetaddressspace": {
            "defaultValue": "192.168.0.0/16",
            "type": "String",
            "metadata": {
                "description": "Address space for virtual network"
            }
        },
        "vnetsubnetprefix": {
            "defaultValue": "192.168.0.0/24",
            "type": "String",
            "metadata": {
                "description": "Subnet prefix for virtual network"
            }
        },
        "natgatewayname": {
            "defaultValue": "myNATgateway",
            "type": "String",
            "metadata": {
              "description": "Name of the NAT gateway"
            }
        },
        "networkinterfacename": {
            "defaultValue": "myvmNIC",
            "type": "String",
            "metadata": {
              "description": "Name of the virtual machine nic"
            }
        },
        "publicipname": {
            "defaultValue": "myPublicIP",
            "type": "String",
            "metadata": {
              "description": "Name of the NAT gateway public IP"
            }
        },
        "nsgname": {
            "defaultValue": "myVMnsg",
            "type": "String",
            "metadata": {
              "description": "Name of the virtual machine NSG"
            }
        },
        "publicipvmname": {
            "defaultValue": "myPublicIPVM",
            "type": "String",
            "metadata": {
              "description": "Name of the virtual machine public IP"
            }
        },
        "publicipprefixname": {
            "defaultValue": "myPublicIPPrefix",
            "type": "String",
            "metadata": {
              "description": "Name of the NAT gateway public IP"
            }
        },
        "adminusername": {
            "type": "String",
            "metadata": {
              "description": "Administrator username for virtual machine"
            }
        },
        "adminpassword": {
            "type": "secureString",
            "metadata": {
              "description": "Administrator password for virtual machine"
            }
        },
        "location": {
            "defaultValue": "[resourceGroup().location]",
            "type": "String",
            "metadata": {
              "description": "Name of resource group"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-11-01",
            "name": "[parameters('nsgname')]",
            "location": "[parameters('location')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "SSH",
                        "properties": {
                            "protocol": "TCP",
                            "sourcePortRange": "*",
                            "destinationPortRange": "22",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 300,
                            "direction": "Inbound"
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-11-01",
            "name": "[parameters('publicipname')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4
            }
        },
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-11-01",
            "name": "[parameters('publicipvmname')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4
            }
        },
        {
            "type": "Microsoft.Network/publicIPPrefixes",
            "apiVersion": "2019-11-01",
            "name": "[parameters('publicipprefixname')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "prefixLength": 31,
                "publicIPAddressVersion": "IPv4"
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2019-07-01",
            "name": "[parameters('vmname')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkinterfacename'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('vmsize')]"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "Canonical",
                        "offer": "UbuntuServer",
                        "sku": "18.04-LTS",
                        "version": "latest"
                    },
                    "osDisk": {
                        "osType": "Linux",
                        "name": "[concat(parameters('vmname'), '_disk1')]",
                        "createOption": "FromImage",
                        "caching": "ReadWrite",
                        "managedDisk": {
                            "storageAccountType": "Premium_LRS"   
                        },
                        "diskSizeGB": 30
                    }

                },
                "osProfile": {
                    "computerName": "[parameters('vmname')]",
                    "adminUsername": "[parameters('adminusername')]",
                    "adminPassword": "[parameters('adminpassword')]",
                    "linuxConfiguration": {
                        "disablePasswordAuthentication": false,
                        "provisionVMAgent": true
                    },
                    "allowExtensionOperations": true
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', parameters('networkinterfacename'))]"
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2019-11-01",
            "name": "[parameters('vnetname')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natgatewayname'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[parameters('vnetaddressspace')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[parameters('subnetname')]",
                        "properties": {
                            "addressPrefix": "[parameters('vnetsubnetprefix')]",
                            "natGateway": {
                                "id": "[resourceId('Microsoft.Network/natGateways', parameters('natgatewayname'))]"
                            },
                            "privateEndpointNetworkPolicies": "Enabled",
                            "privateLinkServiceNetworkPolicies": "Enabled"
                        }
                    }
                ],
                "enableDdosProtection": false,
                "enableVmProtection": false
            }
        },
        {
            "type": "Microsoft.Network/natGateways",
            "apiVersion": "2019-11-01",
            "name": "[parameters('natgatewayname')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicipname'))]",
                "[resourceId('Microsoft.Network/publicIPPrefixes', parameters('publicipprefixname'))]"
            ],
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "idleTimeoutInMinutes": 4,
                "publicIpAddresses": [
                    {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicipname'))]"
                    }
                ],
                "publicIpPrefixes": [
                    {
                        "id": "[resourceId('Microsoft.Network/publicIPPrefixes', parameters('publicipprefixname'))]"
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2019-11-01",
            "name": "[concat(parameters('vnetname'), '/mySubnet')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetname'))]",
                "[resourceId('Microsoft.Network/natGateways', parameters('natgatewayname'))]"
            ],
            "properties": {
                "addressPrefix": "[parameters('vnetsubnetprefix')]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natgatewayname'))]"
                },
                "privateEndpointNetworkPolicies": "Enabled",
                "privateLinkServiceNetworkPolicies": "Enabled"
            }
        },
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2019-11-01",
            "name": "[parameters('networkinterfacename')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicipvmname'))]",
                "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetname'), 'mySubnet')]",
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgname'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAddress": "192.168.0.4",
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicipvmname'))]"
                            },
                            "subnet": {
                                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('vnetname'), 'mySubnet')]"
                            },
                            "primary": true,
                            "privateIPAddressVersion": "IPv4"
                        }
                    }
                ],
                "enableAcceleratedNetworking": false,
                "enableIPForwarding": false,
                "networkSecurityGroup": {
                    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgname'))]"
                }
            }
        }
    ]
}
```

该模板中定义了九个 Azure 资源：

<!--Not Availble on below tempalate link of Global document site-->

**Microsoft.Network**

* **Microsoft.Network/natGateways**：创建 NAT 网关资源。

* **Microsoft.Network/networkSecurityGroups**：创建网络安全组。

    * **Microsoft.Network/networkSecurityGroups/securityRules**：创建安全规则。

* **Microsoft.Network/publicIPAddresses**：创建公共 IP 地址。

* **Microsoft.Network/publicIPPrefixes**：创建公共 IP 前缀。

* **Microsoft.Network/virtualNetworks**：创建虚拟网络。

    * **Microsoft.Network/virtualNetworks/subnets**：创建虚拟网络子网。

* **Microsoft.Network/networkinterfaces**：创建网络接口。

**Microsoft.Compute**

* **Microsoft.Compute/virtualMachines**：创建虚拟机。

### <a name="deploy-the-template"></a>部署模板

**Azure CLI**

```azurecli
read -p "Enter the location (i.e. chinaeast): " location
resourceGroupName="myResourceGroupNAT"
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json" 

az group create \
--name $resourceGroupName \
--location $location

az group deployment create \
--resource-group $resourceGroupName \
--template-uri  $templateUri
```

**Azure PowerShell**

```powershell
$location = Read-Host -Prompt "Enter the location (i.e. chinaeast)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-nat-gateway-1-vm/azuredeploy.json"

$resourceGroupName = "myResourceGroupNAT"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri
```

**Azure 门户**

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-nat-gateway-1-vm%2Fazuredeploy.json)

## <a name="review-deployed-resources"></a>查看已部署的资源

1. 登录到 [Azure 门户](https://portal.azure.cn)。

2. 从左侧窗格中选择“资源组”。

3. 选择你在上一部分中创建的资源组。 默认资源组名称是 **myResourceGroupNAT**

4. 验证是否在资源组中创建了以下资源：

    :::image type="content" source="./media/quick-create-template/nat-gateway-template-rg.png" alt-text="部署到 Azure":::

## <a name="clean-up-resources"></a>清理资源

**Azure CLI**

如果不再需要上述资源组及其包含的所有资源，可以使用 [az group delete](https://docs.azure.cn/cli/group#az-group-delete) 命令将其删除。

```azurecli 
  az group delete \
    --name myResourceGroupNAT
```

**Azure PowerShell**

如果不再需要上述资源组及其包含的所有资源，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=latest) 命令将其删除。

```powershell 
Remove-AzResourceGroup -Name myResourceGroupNAT
```

**Azure 门户**

如果不再需要上述资源组、NAT 网关和所有相关资源，请将其删除。 选择包含 NAT 网关的资源组 **myResourceGroupNAT**，然后选择“删除”。

## <a name="next-steps"></a>后续步骤

在本快速入门中，我们创建了：

* NAT 网关资源
* 虚拟网络
* Ubuntu 虚拟机

虚拟机部署到与 NAT 网关关联的虚拟网络子网。 

若要详细了解虚拟网络 NAT 和 Azure 资源管理器，请继续阅读以下文章。

* 阅读[虚拟网络 NAT 概述](nat-overview.md)
* 了解 [NAT 网关资源](nat-gateway-resource.md)
* 了解有关 [Azure 资源管理器](../azure-resource-manager/management/overview.md)的详细信息

<!-- Update_Description: update meta properties, wording update, update link -->