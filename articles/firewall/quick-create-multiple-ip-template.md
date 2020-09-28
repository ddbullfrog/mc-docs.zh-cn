---
title: 快速入门 - 创建具有多个公共 IP 地址的 Azure 防火墙 - 资源管理器模板
description: 本快速入门介绍了如何使用 Azure 资源管理器模板（ARM 模板）创建具有多个公共 IP 地址的 Azure 防火墙。
services: firewall
ms.service: firewall
ms.topic: quickstart
ms.custom: subject-armqs
origin.date: 08/28/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: yes
ms.testdate: 08/03/2020
ms.author: v-yeche
ms.openlocfilehash: 47f5342798064ad163f3b53690e11df7714e87ec
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246796"
---
<!--Verified successfully on 05/25/2020-->
<!--Successfully on Mooncake portal-->
# <a name="quickstart-create-an-azure-firewall-with-multiple-public-ip-addresses---arm-template"></a>快速入门：创建具有多个公共 IP 地址的 Azure 防火墙 - ARM 模板

在本快速入门中，使用 Azure 资源管理器模板（ARM 模板）部署具有多个公共 IP 地址的 Azure 防火墙。 部署的防火墙具有 NAT 规则收集规则，这些规则允许通过 RDP 连接与两个 Windows Server 2019 虚拟机进行连接。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

若要详细了解具有多个公共 IP 地址的 Azure 防火墙，请参阅[使用 Azure PowerShell 部署具有多个公共 IP 地址的 Azure 防火墙](deploy-multi-public-ip-powershell.md)。

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

- 具有活动订阅的 Azure 帐户。 [免费创建帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="review-the-template"></a>查看模板

此模板创建具有两个公共 IP 地址的 Azure 防火墙，以及用于支持 Azure 防火墙的必要资源。

本快速入门中使用的模板来自 [Azure 快速启动模板](https://github.com/Azure/azure-quickstart-templates/tree/master/fw-docs-qs)。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "adminUsername": {
            "type": "String",
            "metadata": {
                "description": "Admin username for the backend servers"
            }
        },
        "adminPassword": {
            "type": "SecureString",
            "metadata": {
                "description": "Password for the admin account on the backend servers"
            }
        },
        "location": {
            "defaultValue": "[resourceGroup().location]",
            "type": "String",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "vmSize": {
            "defaultValue": "Standard_B2ms",
            "type": "String",
            "metadata": {
                "description": "Size of the virtual machine."
            }
        }
    },
    "variables": {
        "virtualMachines_myVM_name": "myVM",
        "virtualNetworks_myVNet_name": "myVNet",
        "net_interface": "net-int",
        "ipconfig_name": "ipconfig",
        "publicIPAddress": "public_ip",
        "nsg_name": "vm-nsg",
        "firewall_name": "FW-01",
        "vnet_prefix": "10.0.0.0/16",
        "fw_subnet_prefix": "10.0.0.0/24",
        "backend_subnet_prefix": "10.0.1.0/24",
        "azureFirewallSubnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets',variables('virtualNetworks_myVNet_name'), 'AzureFirewallSubnet')]",
        "azureFirewallSubnetJSON": "[json(format('{{\"id\": \"{0}\"}}', variables('azureFirewallSubnetId')))]",
        "copy": [
            {
                "name": "azureFirewallIpConfigurations",
                "count": 2,
                "input": {
                    "name": "[concat('IpConf', copyIndex('azureFirewallIpConfigurations',1))]",
                    "properties": {
                        "subnet": "[if(equals(copyIndex('azureFirewallIpConfigurations',1), 1), variables('azureFirewallSubnetJSON'), json('null'))]",
                        "publicIPAddress": {
                            "id": "[resourceId('Microsoft.Network/publicIPAddresses', concat(variables('publicIPAddress'), copyIndex('azureFirewallIpConfigurations',1)))]"
                        }
                    }
                }
            }
        ]
    },
    "resources": [
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-11-01",
            "name": "[concat(variables('nsg_name'), copyIndex(1))]",
            "location": "[parameters('location')]",
            "copy": {
                "name": "nsg-loop",
                "count": 2
            },
            "properties": {
                "securityRules": [
                    {
                        "name": "RDP",
                        "properties": {
                            "protocol": "TCP",
                            "sourcePortRange": "*",
                            "destinationPortRange": "3389",
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
            "name": "[concat(variables('publicIPAddress'), copyIndex(1))]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "copy": {
                "name": "publicip-loop",
                "count": 2
            },
            "properties": {
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Static",
                "idleTimeoutInMinutes": 4
            }

        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2019-11-01",
            "name": "[variables('virtualNetworks_myVNet_name')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/routeTables', 'rt-01')]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('vnet_prefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "myBackendSubnet",
                        "properties": {
                            "addressPrefix": "[variables('backend_subnet_prefix')]",
                            "routeTable": {
                                "id": "[resourceId('Microsoft.Network/routeTables', 'rt-01')]"
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
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2019-11-01",
            "name": "[concat(variables('virtualNetworks_myVNet_name'), '/AzureFirewallSubnet')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworks_myVNet_name'))]"
            ],
            "properties": {
                "addressPrefix": "[variables('fw_subnet_prefix')]",
                "privateEndpointNetworkPolicies": "Enabled",
                "privateLinkServiceNetworkPolicies": "Enabled"
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2019-07-01",
            "name": "[concat(variables('virtualMachines_myVM_name'), copyIndex(1))]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('net_interface'), copyIndex(1)))]"
            ],
            "copy": {
                "name": "vm-loop",
                "count": 2
            },
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('vmSize')]"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "MicrosoftWindowsServer",
                        "offer": "WindowsServer",
                        "sku": "2019-Datacenter",
                        "version": "latest"
                    },
                    "osDisk": {
                        "osType": "Windows",
                        "createOption": "FromImage",
                        "caching": "ReadWrite",
                        "managedDisk": {
                            "storageAccountType": "StandardSSD_LRS"
                        },
                        "diskSizeGB": 127
                    }
                },
                "osProfile": {
                    "computerName": "[concat(variables('virtualMachines_myVM_name'), copyIndex(1))]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]",
                    "windowsConfiguration": {
                        "provisionVMAgent": true,
                        "enableAutomaticUpdates": true
                    },
                    "allowExtensionOperations": true
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces', concat(variables('net_interface'), copyIndex(1)))]"
                        }
                    ]
                }
            }

        },
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2019-11-01",
            "name": "[concat(variables('net_interface'), copyIndex(1))]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworks_myVNet_name'))]",
                "[resourceId('Microsoft.Network/networkSecurityGroups', concat(variables('nsg_name'), copyIndex(1)))]"
            ],
            "copy": {
                "name": "int-loop",
                "count": 2
            },
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "[concat(variables('ipconfig_name'), copyIndex(1))]",
                        "properties": {
                            "subnet": {
                                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworks_myVNet_name'), 'myBackendSubnet')]"
                            },
                            "primary": true
                        }
                    }
                ],
                "enableAcceleratedNetworking": false,
                "enableIPForwarding": false,
                "networkSecurityGroup": {
                    "id": "[resourceId('Microsoft.Network/networkSecurityGroups', concat(variables('nsg_name'), copyIndex(1)))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/azureFirewalls",
            "apiVersion": "2019-11-01",
            "name": "[variables('firewall_name')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', concat(variables('publicIPAddress'), 1))]",
                "[resourceId('Microsoft.Network/publicIPAddresses', concat(variables('publicIPAddress'), 2))]",
                "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworks_myVNet_name'), 'AzureFirewallSubnet')]"

            ],
            "properties": {
                "sku": {
                    "name": "AZFW_VNet",
                    "tier": "Standard"
                },
                "threatIntelMode": "Alert",
                "ipConfigurations": "[variables('azureFirewallIpConfigurations')]",
                "applicationRuleCollections": [
                    {
                        "name": "web",
                        "properties": {
                            "priority": 100,
                            "action": {
                                "type": "Allow"
                            },
                            "rules": [
                                {
                                    "name": "wan-address",
                                    "protocols": [
                                        {
                                            "protocolType": "Http",
                                            "port": 80
                                        },
                                        {
                                            "protocolType": "Https",
                                            "port": 443
                                        }
                                    ],
                                    "targetFqdns": [
                                        "getmywanip.com"
                                    ],
                                    "sourceAddresses": [
                                        "*"
                                    ]
                                },
                                {
                                    "name": "google",
                                    "protocols": [
                                        {
                                            "protocolType": "Http",
                                            "port": 80
                                        },
                                        {
                                            "protocolType": "Https",
                                            "port": 443
                                        }
                                    ],
                                    "targetFqdns": [
                                        "www.google.com"
                                    ],
                                    "sourceAddresses": [
                                        "10.0.1.0/24"
                                    ]
                                },
                                {
                                    "name": "wupdate",
                                    "protocols": [
                                        {
                                            "protocolType": "Http",
                                            "port": 80
                                        },
                                        {
                                            "protocolType": "Https",
                                            "port": 443
                                        }
                                    ],
                                    "fqdnTags": [
                                        "WindowsUpdate"
                                    ],
                                    "sourceAddresses": [
                                        "*"
                                    ]
                                }
                            ]
                        }
                    }
                ],
                "natRuleCollections": [
                    {
                        "name": "Coll-01",
                        "properties": {
                            "priority": 100,
                            "action": {
                                "type": "Dnat"
                            },
                            "rules": [
                                {
                                    "name": "rdp-01",
                                    "protocols": [
                                        "TCP"
                                    ],
                                    "translatedAddress": "10.0.1.4",
                                    "translatedPort": "3389",
                                    "sourceAddresses": [
                                        "*"
                                    ],
                                    "destinationAddresses": [ "[reference(resourceId('Microsoft.Network/publicIPAddresses/', concat(variables('publicIPAddress'), 1))).ipAddress]" ],
                                    "destinationPorts": [
                                        "3389"
                                    ]
                                },
                                {
                                    "name": "rdp-02",
                                    "protocols": [
                                        "TCP"
                                    ],
                                    "translatedAddress": "10.0.1.5",
                                    "translatedPort": "3389",
                                    "sourceAddresses": [
                                        "*"
                                    ],
                                    "destinationAddresses": [ "[reference(resourceId('Microsoft.Network/publicIPAddresses/', concat(variables('publicIPAddress'), 2))).ipAddress]" ],
                                    "destinationPorts": [
                                        "3389"
                                    ]
                                }
                            ]
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/routeTables",
            "apiVersion": "2019-11-01",
            "name": "rt-01",
            "location": "[parameters('location')]",
            "properties": {
                "disableBgpRoutePropagation": false,
                "routes": [
                    {
                        "name": "fw",
                        "properties": {
                            "addressPrefix": "0.0.0.0/0",
                            "nextHopType": "VirtualAppliance",
                            "nextHopIpAddress": "10.0.0.4"
                        }
                    }
                ]
            }
        }

    ]
}
```

模板中定义了多个 Azure 资源：

- **Microsoft.Network/publicIPAddresses**
- **Microsoft.Network/networkSecurityGroups**
- **Microsoft.Network/virtualNetworks**
- Microsoft.Compute/virtualMachines 
- **Microsoft.Network/networkInterfaces**
- **Microsoft.Storage/storageAccounts**
- **Microsoft.Network/azureFirewalls**
- **Microsoft.Network/routeTables**

<!--Not Available on Microsoft Template Link-->

## <a name="deploy-the-template"></a>部署模板

将 ARM 模板部署到 Azure：

1. 选择“部署到 Azure”，登录到 Azure 并打开模板。 该模板将创建 Azure 防火墙、网络基础结构和两个虚拟机。

    [:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="部署到 Azure":::](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Ffw-docs-qs%2Fazuredeploy.json)

2. 在门户中的“创建具有多个公共 IP 地址的 Azure 防火墙”页上，键入或选择以下值：
    - 订阅：从现有订阅中选择 
    - 资源组：从现有资源组中选择，或者选择“新建”，然后选择“确定”。 
    - 位置：选择一个位置
    - 管理员用户名：键入管理员用户帐户的用户名 
    - 管理员密码：键入管理员密码或密钥

3. 选择“我同意上述条款和条件”，然后选择“购买”   。 部署可能需要 10 分钟或更长时间才能完成。

## <a name="validate-the-deployment"></a>验证部署

在 Azure 门户中，查看已部署的资源。 记下防火墙公共 IP 地址。  

使用“远程桌面连接”连接到防火墙公共 IP 地址。 成功的连接演示了允许连接到后端服务器的防火墙 NAT 规则。

## <a name="clean-up-resources"></a>清理资源

如果不再需要为防火墙创建的资源，请删除资源组。 这会删除该防火墙和所有相关资源。

若要删除资源组，请调用 `Remove-AzResourceGroup` cmdlet：

```powershell
Remove-AzResourceGroup -Name "<your resource group name>"
```

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [教程：使用 Azure 门户在混合网络中部署和配置 Azure 防火墙](tutorial-hybrid-portal.md)

<!-- Update_Description: update meta properties, wording update, update link -->