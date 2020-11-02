---
title: 快速入门：创建自动化帐户 - Azure 模板
titleSuffix: Azure Automation
description: 本快速入门介绍如何使用 Azure 资源管理器模板创建自动化帐户。
services: automation
documentationcenter: na
author: WenJason
Customer intent: I want to create an Automation account by using an Azure Resource Manager template so that I can automate processes with runbooks.
ms.service: automation
ms.devlang: na
ms.topic: quickstart
ms.workload: infrastructure-services
origin.date: 07/23/2020
ms.date: 11/02/2020
ms.author: v-jay
ms.custom: mvc,subject-armqs
ms.openlocfilehash: b8377ef0ad4b742bf71263bd2a7eb12ea01d710d
ms.sourcegitcommit: ca5e5792f3c60aab406b7ddbd6f6fccc4280c57e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92750210"
---
# <a name="quickstart-create-an-automation-account-by-using-arm-template"></a>快速入门：使用 ARM 模板创建自动化帐户

Azure 自动化提供基于云的自动化和配置服务，用于支持 Azure 环境和非 Azure 环境之间的一致管理。 本快速入门介绍如何部署用于创建自动化帐户的 Azure 资源管理器模板（ARM 模板）。 与其他部署方法相比，使用 ARM 模板需要的步骤更少。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-automation%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://wd.azure.cn/pricing/1rmb-trial-full/?form-type=identityauth)。

## <a name="review-the-template"></a>查看模板

本示例模板执行以下操作：

* 自动创建 Azure Monitor Log Analytics 工作区。
* 自动创建 Azure 自动化帐户。
* 将自动化帐户链接到 Log Analytics 工作区。
* 将示例自动化 runbook 添加到帐户。

>[!NOTE]
>使用 ARM 模板时，不支持创建自动化运行方式帐户。 若要从门户或使用 PowerShell 手动创建运行方式帐户，请参阅[管理运行方式帐户](manage-runas-account.md)。

完成这些步骤后，你需要为自动化帐户[配置诊断设置](automation-manage-send-joblogs-log-analytics.md)，以将 runbook 作业状态和作业流发送到链接的 Log Analytics 工作区。

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-automation/)。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "string",
            "metadata": {
                "description": "Workspace name"
            }
        },
        "sku": {
            "type": "string",
            "allowedValues": [
                "pergb2018",
                "Free",
                "Standalone",
                "PerNode",
                "Standard",
                "Premium"
            ],
            "defaultValue": "pergb2018",
            "metadata": {
                "description": "Pricing tier: perGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium), which are not available to all customers."
            }
        },
        "dataRetention": {
            "type": "int",
            "defaultValue": 30,
            "minValue": 7,
            "maxValue": 730,
            "metadata": {
                "description": "Number of days to retain data."
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Specifies the location in which to create the workspace."
            }
        },
        "automationAccountName": {
            "type": "string",
            "metadata": {
                "description": "Automation account name"
            }
        },
        "automationAccountLocation": {
            "type": "string",
            "metadata": {
                "description": "Specifies the location in which to create the Automation account."
            }
        },
        "sampleGraphicalRunbookName": {
            "type": "String",
            "defaultValue": "AzureAutomationTutorial"
        },
        "sampleGraphicalRunbookDescription": {
            "type": "String",
            "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
        },
        "samplePowerShellRunbookName": {
            "type": "String",
            "defaultValue": "AzureAutomationTutorialScript"
        },
        "samplePowerShellRunbookDescription": {
            "type": "String",
            "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
        },
        "samplePython2RunbookName": {
            "type": "String",
            "defaultValue": "AzureAutomationTutorialPython2"
        },
        "samplePython2RunbookDescription": {
            "type": "String",
            "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
        },
        "_artifactsLocation": {
            "type": "string",
            "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-automation/",
            "metadata": {
                "description": "URI to artifacts location"
            }
        },
        "_artifactsLocationSasToken": {
            "type": "securestring",
            "defaultValue": "",
            "metadata": {
                "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "apiVersion": "2020-03-01-preview",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "name": "[parameters('sku')]"
                },
                "retentionInDays": "[parameters('dataRetention')]",
                "features": {
                    "searchVersion": 1,
                    "legacy": 0
                }
            }
        },
        {
            "type": "Microsoft.Automation/automationAccounts",
            "apiVersion": "2020-01-13-preview",
            "name": "[parameters('automationAccountName')]",
            "location": "[parameters('automationAccountLocation')]",
            "dependsOn": [
                "[parameters('workspaceName')]"
            ],
            "properties": {
                "sku": {
                    "name": "Basic"
                }
            },
            "resources": [
                {
                    "type": "runbooks",
                    "apiVersion": "2018-06-30",
                    "name": "[parameters('sampleGraphicalRunbookName')]",
                    "location": "[parameters('automationAccountLocation')]",
                    "dependsOn": [
                        "[parameters('automationAccountName')]"
                    ],
                    "properties": {
                        "runbookType": "GraphPowerShell",
                        "logProgress": "false",
                        "logVerbose": "false",
                        "description": "[parameters('sampleGraphicalRunbookDescription')]",
                        "publishContentLink": {
                            "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.graphrunbook', parameters('_artifactsLocationSasToken')))]",
                            "version": "1.0.0.0"
                        }
                    }
                },
                {
                    "type": "runbooks",
                    "apiVersion": "2018-06-30",
                    "name": "[parameters('samplePowerShellRunbookName')]",
                    "location": "[parameters('automationAccountLocation')]",
                    "dependsOn": [
                        "[parameters('automationAccountName')]"
                    ],
                    "properties": {
                        "runbookType": "PowerShell",
                        "logProgress": "false",
                        "logVerbose": "false",
                        "description": "[parameters('samplePowerShellRunbookDescription')]",
                        "publishContentLink": {
                            "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.ps1', parameters('_artifactsLocationSasToken')))]",
                            "version": "1.0.0.0"
                        }
                    }
                },
                {
                    "type": "runbooks",
                    "apiVersion": "2018-06-30",
                    "name": "[parameters('samplePython2RunbookName')]",
                    "location": "[parameters('automationAccountLocation')]",
                    "dependsOn": [
                        "[parameters('automationAccountName')]"
                    ],
                    "properties": {
                        "runbookType": "Python2",
                        "logProgress": "false",
                        "logVerbose": "false",
                        "description": "[parameters('samplePython2RunbookDescription')]",
                        "publishContentLink": {
                            "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorialPython2.py', parameters('_artifactsLocationSasToken')))]",
                            "version": "1.0.0.0"
                        }
                    }
                }
            ]
        },
        {
            "type": "Microsoft.OperationalInsights/workspaces/linkedServices",
            "apiVersion": "2020-03-01-preview",
            "name": "[concat(parameters('workspaceName'), '/' , 'Automation')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[parameters('workspaceName')]",
                "[parameters('automationAccountName')]"
            ],
            "properties": {
                "resourceId": "[resourceId('Microsoft.Automation/automationAccounts', parameters('automationAccountName'))]"
            }
        }
    ]
}
```

### <a name="api-versions"></a>API 版本

下表列出了此示例中使用的资源的 API 版本。

| 资源 | 资源类型 | API 版本 |
|:---|:---|:---|
| [工作区](https://docs.microsoft.com/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 2020-03-01-preview |
| [自动化帐户](https://docs.microsoft.com/azure/templates/microsoft.automation/automationaccounts) | automation | 2020-01-13-preview |
| [工作区关联的服务](https://docs.microsoft.com/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 2020-03-01-preview |

### <a name="before-you-use-the-template"></a>使用模版之前

已为你配置 JSON 参数模板，该模板用于指定以下项：

* 工作区的名称。
* 要在其中创建工作区的区域。
* 自动化帐户的名称。
* 要在其中创建自动化帐户的区域。

模板中的以下参数设置为 Log Analytics 工作区的默认值：

* sku 默认为“按 GB”定价层，该层已在 2018 年 4 月的定价模型中发布。
* dataRetention 默认为 30 天。

>[!WARNING]
>如果要在订阅中创建或配置 Log Analytics 工作区，而该订阅已加入 2018 年 4 月的定价模型，则唯一有效的 Log Analytics 定价层为 PerGB2018。
>

JSON 模板为其他参数指定默认值，这些参数将会用作环境中的标准配置。 可以将模板存储在 Azure 存储帐户中，以便在组织中共享访问。 有关使用模板的详细信息，请参阅[使用 ARM 模板和 Azure CLI 部署资源](../azure-resource-manager/templates/deploy-cli.md)。

如果你不熟悉 Azure 自动化和 Azure Monitor，请务必了解以下配置详细信息。 当你尝试创建、配置和使用链接到新自动化帐户的 Log Analytics 工作区时，这些信息有助于避免出错。

* 查看[其他详细信息](../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace)以充分了解工作区配置选项，如访问控制模式、定价层、保留期和产能预留级别。

* 查看[工作区映射](how-to/region-mappings.md)，以内联方式或在参数文件中指定支持的区域。 只有某些区域支持链接订阅中的 Log Analytics 工作区和自动化帐户。

* 如果刚开始使用 Azure Monitor 日志，并且尚未部署工作区，则应查看[工作区设计指南](../azure-monitor/platform/design-logs-deployment.md)。 它将有助于了解访问控制以及建议组织采用的设计实现策略。

## <a name="deploy-the-template"></a>部署模板

1. 选择下图登录到 Azure 并打开一个模板。 该模板将创建一个 Azure 自动化帐户和一个 Log Analytics 工作区，并将自动化帐户关联到工作区。

    [![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-automation%2Fazuredeploy.json)

2. 输入相应的值。

3. 部署可能需要几分钟才能完成。 完成后，输出类似于以下内容：

    ![部署完成后的示例结果](media/quickstart-create-automation-account-template/template-output.png)

## <a name="review-deployed-resources"></a>查看已部署的资源

1. 登录 [Azure 门户](https://portal.azure.cn)。

2. 在 Azure 门户中，打开刚刚创建的自动化帐户。 

3. 在左侧窗格中，选择“Runbook”。 “Runbook”页上列出了三个使用自动化帐户创建的教程 runbook。

    ![使用自动化帐户创建的教程 runbook](./media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. 在左侧窗格中，选择“关联的工作区”。 “关联的工作区”页上显示了先前指定的、已关联到自动化帐户的 Log Analytics 工作区。

    ![已关联到 Log Analytics 工作区的自动化帐户](./media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

## <a name="clean-up-resources"></a>清理资源

如果不再需要它们，请取消与 Log Analytics 工作区的自动化帐户的关联，然后删除自动化帐户和工作区。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了一个自动化帐户和一个 Log Analytics 工作区，并将它们关联在一起。

若要了解详细信息，请继续学习 Azure 自动化教程。

> [!div class="nextstepaction"]
> [Azure 自动化教程](learn/automation-tutorial-runbook-graphical.md)