---
title: 快速入门：使用模板创建 Interactive Query 群集 - Azure HDInsight
description: 本快速入门介绍如何使用 Azure 资源管理器模板在 Azure HDInsight 中创建一个 Interactive Query 群集。
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/13/2020
ms.openlocfilehash: 56d544d569d89a468d0300eab6d1ba8f63af4932
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472608"
---
# <a name="quickstart-create-interactive-query-cluster-in-azure-hdinsight-using-arm-template"></a>快速入门：使用 ARM 模板在 Azure HDInsight 中创建 Interactive Query 群集

本快速入门将使用 Azure 资源管理器模板（ARM 模板）在 Azure HDInsight 中创建一个 [Interactive Query](./apache-interactive-query-get-started.md) 群集。 交互式查询（也称为 Apache Hive LLAP 或[低延迟分析处理](https://cwiki.apache.org/confluence/display/Hive/LLAP)）是一种 Azure HDInsight [群集类型](../hdinsight-hadoop-provision-linux-clusters.md#cluster-type)。

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[![部署到 Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-interactive-hive%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-hdinsight-interactive-hive/)。

```Json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "metadata": {
        "description": "The name of the HDInsight cluster to create."
      }
    },
    "clusterLoginUserName": {
      "type": "string",
      "metadata": {
        "description": "These credentials can be used to submit jobs to the cluster and to log into cluster dashboards."
      }
    },
    "clusterLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "sshUserName": {
      "type": "string",
      "metadata": {
        "description": "These credentials can be used to remotely access the cluster."
      }
    },
    "sshPassword": {
      "type": "securestring",
      "metadata": {
        "description": "The password must be at least 10 characters in length and must contain at least one digit, one non-alphanumeric character, and one upper or lower case letter."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "defaultStorageAccount": {
      "name": "[uniqueString(resourceGroup().id)]",
      "type": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('defaultStorageAccount').name]",
      "location": "[parameters('location')]",
      "apiVersion": "2016-01-01",
      "sku": {
        "name": "[variables('defaultStorageAccount').type]"
      },
      "kind": "Storage",
      "properties": {}
    },
    {
      "type": "Microsoft.HDInsight/clusters",
      "name": "[parameters('clusterName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2015-03-01-preview",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/',variables('defaultStorageAccount').name)]"
      ],
      "properties": {
        "clusterVersion": "3.6",
        "osType": "Linux",
        "tier": "Standard",
        "clusterDefinition": {
          "kind": "interactivehive",
          "configurations": {
            "gateway": {
              "restAuthCredential.isEnabled": true,
              "restAuthCredential.username": "[parameters('clusterLoginUserName')]",
              "restAuthCredential.password": "[parameters('clusterLoginPassword')]"
            }
          }
        },
        "storageProfile": {
          "storageaccounts": [
            {
              "name": "[replace(replace(concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('defaultStorageAccount').name), '2016-01-01').primaryEndpoints.blob),'https:',''),'/','')]",
              "isDefault": true,
              "container": "[parameters('clusterName')]",
              "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name), '2016-01-01').keys[0].value]"
            }
          ]
        },
        "computeProfile": {
          "roles": [
            {
              "name": "headnode",
              "minInstanceCount": 1,
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D13_V2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              },
              "virtualNetworkProfile": null,
              "scriptActions": []
            },
            {
              "name": "workernode",
              "minInstanceCount": 1,
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D14_v2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              },
              "virtualNetworkProfile": null,
              "scriptActions": []
            },
            {
              "name": "zookeepernode",
              "minInstanceCount": 1,
              "targetInstanceCount": 3,
              "hardwareProfile": {
                "vmSize": "Large"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              },
              "virtualNetworkProfile": null,
              "scriptActions": []
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "storage": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', variables('defaultStorageAccount').name))]"
    },
    "cluster": {
      "type": "object",
      "value": "[reference(resourceId('Microsoft.HDInsight/clusters',parameters('clusterName')))]"
    }
  }
}
```

该模板中定义了两个 Azure 资源：

* [Microsoft.Storage/storageAccounts](https://docs.microsoft.com/azure/templates/microsoft.storage/storageaccounts)：创建 Azure 存储帐户。
* [Microsoft.HDInsight/cluster](https://docs.microsoft.com/azure/templates/microsoft.hdinsight/clusters)：创建 HDInsight 群集。

### <a name="deploy-the-template"></a>部署模板

1. 选择下面的“部署到 Azure”按钮以登录到 Azure，并打开资源管理器模板。

    [![部署到 Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-interactive-hive%2Fazuredeploy.json)

1. 输入或选择下列值：

    |属性 |说明 |
    |---|---|
    |订阅|从下拉列表中选择用于此群集的 Azure 订阅。|
    |资源组|从下拉列表中选择现有资源组，或选择“新建”。|
    |位置|将使用用于资源组的位置自动填充此值。|
    |群集名称|输入任何全局唯一的名称。 对于此模板，请只使用小写字母和数字。|
    |群集登录用户名|提供用户名，默认值为 **admin** 。|
    |群集登录密码|提供密码。 密码长度不得少于 10 个字符，且至少必须包含一个数字、一个大写字母和一个小写字母、一个非字母数字字符（' " ` 字符除外）。 |
    |SSH 用户名|提供用户名，默认值为 sshuser|
    |SSH 密码|提供密码。|

    ![资源管理器模板 HBase](./media/quickstart-resource-manager-template/resource-manager-template-hive.png)

1. 查看“条款和条件”。 接着选择“我同意上述条款和条件”，然后选择“购买” 。 你会收到一则通知，说明正在进行部署。 创建群集大约需要 20 分钟时间。

## <a name="review-deployed-resources"></a>查看已部署的资源

创建群集后，你会收到“部署成功”通知，通知中附有“转到资源”链接 。 “资源组”页会列出新的 HDInsight 群集以及与该群集关联的默认存储。 每个群集都有一个 [Azure Blob 存储](../hdinsight-hadoop-use-blob-storage.md)帐户或一个 [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md) 依赖项。 该帐户称为默认存储帐户。 HDInsight 群集及其默认存储帐户必须共存于同一个 Azure 区域中。 删除群集不会删除存储帐户。

## <a name="clean-up-resources"></a>清理资源

完成本快速入门后，可以删除群集。 有了 HDInsight，便可以将数据存储在 Azure 存储中，因此可以在群集不用时安全地删除群集。 此外，还需要为 HDInsight 群集付费，即使不用也是如此。 由于群集费用数倍于存储空间费用，因此在群集不用时删除群集可以节省费用。

从 Azure 门户导航到群集，然后选择“删除”。

![资源管理器模板 HBase](./media/quickstart-resource-manager-template/azure-portal-delete-hive.png)

还可以选择资源组名称来打开“资源组”页，然后选择“删除资源组”。 通过删除资源组，可以删除 HDInsight 群集和默认存储帐户。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解了如何使用资源管理器模板在 HDInsight 中创建 Interactive Query 群集。 下一篇文章介绍如何使用 Apache Zeppelin 运行 Apache Hive 查询。

> [!div class="nextstepaction"]
> [使用 Apache Zeppelin 在 Azure HDInsight 中执行 Apache Hive 查询](./hdinsight-connect-hive-zeppelin.md)
