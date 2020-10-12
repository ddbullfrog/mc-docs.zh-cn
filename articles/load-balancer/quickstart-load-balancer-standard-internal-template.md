---
title: 使用 Azure 资源管理器模板（ARM 模板）创建内部负载均衡器
description: 了解如何使用 Azure 资源管理器模板（ARM 模板）创建内部 Azure 负载均衡器。
services: load-balancer
author: WenJason
ms.service: load-balancer
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: v-jay
origin.date: 09/14/2020
ms.date: 10/19/2020
ms.openlocfilehash: 3717b44513d3fbf48edd6a778881ad52f1b6abbc
ms.sourcegitcommit: 57511ab990fbb26305a76beee48f0c223963f7ca
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/12/2020
ms.locfileid: "91943556"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-by-using-an-arm-template"></a>快速入门：使用 ARM 模板创建内部负载均衡器，以对 VM 进行负载均衡

本快速入门介绍如何使用 Azure 资源管理器模板（ARM 模板）来创建内部 Azure 负载均衡器。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://wd.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-internal-load-balancer)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password"
      }
    },
    "vmNamePrefix": {
      "type": "string",
      "defaultValue": "BackendVM",
      "metadata": {
        "description": "Prefix to use for VM names"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_DS1_V2",
      "metadata": {
        "description": "Size of the virtual machines"
      }
    }
  },
  "variables": {
    "availabilitySetName": "AvSet",
    "storageAccountType": "Standard_LRS",
    "storageAccountName": "[uniqueString(resourceGroup().id)]",
    "virtualNetworkName": "vNet",
    "subnetName": "backendSubnet",
    "loadBalancerName": "ilb",
    "networkInterfaceName": "nic",
    "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
    "numberOfInstances": 2,
  },
  "resources": [
    {
      "apiVersion": "2019-06-01",
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "kind": "StorageV2",
      "sku": {
        "name": "[variables('storageAccountType')]"
      }
    },
    {
      "apiVersion": "2019-12-01",
      "type": "Microsoft.Compute/availabilitySets",
      "location": "[parameters('location')]",
      "name": "[variables('availabilitySetName')]",
      "properties": {
        "PlatformUpdateDomainCount": 2,
        "PlatformFaultDomainCount": 2
      },
      "sku": {
        "name": "Aligned"
      }
    },
    {
      "apiVersion": "2020-05-01",
      "type": "Microsoft.Network/virtualNetworks",
      "name": "[variables('virtualNetworkName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "10.0.0.0/16"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "10.0.2.0/24"
            }
          }
        ]
      }
    },
    {
      "apiVersion": "2020-05-01",
      "type": "Microsoft.Network/networkInterfaces",
      "name": "[concat(variables('networkInterfaceName'), copyindex())]",
      "location": "[parameters('location')]",
      "copy": {
        "name": "nicLoop",
        "count": "[variables('numberOfInstances')]"
      },
      "dependsOn": [
        "[variables('virtualNetworkName')]",
        "[variables('loadBalancerName')]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
              "subnet": {
                "id": "[variables('subnetRef')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', variables('loadBalancerName'), 'BackendPool1')]"
                }
              ]
            }
          }
        ]
      }
    },
    {
      "apiVersion": "2020-05-01",
      "type": "Microsoft.Network/loadBalancers",
      "name": "[variables('loadBalancerName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[variables('virtualNetworkName')]"
      ],
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "frontendIPConfigurations": [
          {
            "properties": {
              "subnet": {
                "id": "[variables('subnetRef')]"
              },
              "privateIPAddress": "10.0.2.6",
              "privateIPAllocationMethod": "Static"
            },
            "name": "LoadBalancerFrontend"
          }
        ],
        "backendAddressPools": [
          {
            "name": "BackendPool1"
          }
        ],
        "loadBalancingRules": [
          {
            "properties": {
              "frontendIPConfiguration": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIpConfigurations', variables('loadBalancerName'), 'LoadBalancerFrontend')]"
              },
              "backendAddressPool": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', variables('loadBalancerName'), 'BackendPool1')]"
              },
              "probe": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/probes', variables('loadBalancerName'), 'lbprobe')]"
              },
              "protocol": "Tcp",
              "frontendPort": 80,
              "backendPort": 80,
              "idleTimeoutInMinutes": 15
            },
            "Name": "lbrule"
          }
        ],
        "probes": [
          {
            "properties": {
              "protocol": "Tcp",
              "port": 80,
              "intervalInSeconds": 15,
              "numberOfProbes": 2
            },
            "name": "lbprobe"
          }
        ]
      }
    },
    {
      "apiVersion": "2019-12-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[concat(parameters('vmNamePrefix'), copyindex())]",
      "copy": {
        "name": "virtualMachineLoop",
        "count": "[variables('numberOfInstances')]"
      },
      "location": "[parameters('location')]",
      "dependsOn": [
        "[variables('storageAccountName')]",
        "nicLoop",
        "[variables('availabilitySetName')]"
      ],
      "properties": {
        "availabilitySet": {
          "id": "[resourceId('Microsoft.Compute/availabilitySets',variables('availabilitySetName'))]"
        },
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[concat(parameters('vmNamePrefix'), copyIndex())]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('networkInterfaceName'), copyindex()))]"
            }
          ]
        },
        "diagnosticsProfile": {
          "bootDiagnostics": {
            "enabled": true,
            "storageUri": "[reference(variables('storageAccountName')).primaryEndpoints.blob]"
          }
        }
      }
    }
  ]
}
```

该模板中已定义了多个 Azure 资源：

- [**Microsoft.Storage/storageAccounts**](https://docs.microsoft.com/azure/templates/microsoft.storage/storageaccounts)：用于启动诊断的虚拟机存储帐户。
- [**Microsoft.Compute/availabilitySets**](https://docs.microsoft.com/azure/templates/microsoft.compute/availabilitySets)：虚拟机的可用性集。
- [**Microsoft.Network/virtualNetworks**](https://docs.microsoft.com/azure/templates/microsoft.network/virtualNetworks)：负载均衡器和虚拟机的虚拟网络。
- [**Microsoft.Network/networkInterfaces**](https://docs.microsoft.com/azure/templates/microsoft.network/networkInterfaces)：虚拟机的网络接口。
- [**Microsoft.Network/loadBalancers**](https://docs.microsoft.com/azure/templates/microsoft.network/loadBalancers)：内部负载均衡器。
- [**Microsoft.Compute/virtualMachines**](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualMachines)：虚拟机。

## <a name="deploy-the-template"></a>部署模板

**Azure CLI**

```azurecli
read -p "Enter the location (i.e. chinaeast2): " location
$resourceGroupName="myResourceGroupLB"
$templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-2-vms-internal-load-balancer/azuredeploy.json" 

az group create \
--name $resourceGroupName \
--location $location

az deployment group create \
--resource-group $resourceGroupName \
--template-uri  $templateUri
```

## <a name="review-deployed-resources"></a>查看已部署的资源

1. 登录 [Azure 门户](https://portal.azure.cn)。

2. 从左侧窗格中选择“资源组”。

3. 选择你在上一部分中创建的资源组。 默认资源组名称是“myResourceGroupLB”

4. 验证是否在资源组中创建了以下资源：

:::image type="content" source="media/quickstart-load-balancer-standard-internal-template/verify-creation.png" alt-text="用于验证资源创建的用户 Azure 门户。" border="true":::

## <a name="clean-up-resources"></a>清理资源

如果不再需要上述资源组及其包含的所有资源，可以使用 [az group delete](/cli/group#az-group-delete) 命令将其删除。

```azurecli 
  az group delete \
    --name myResourceGroupLB
```

## <a name="next-steps"></a>后续步骤

有关引导你完成模板创建过程的分步教程，请参阅：

> [!div class="nextstepaction"]
> 创建和部署你的第一个 ARM 模板[