---
title: 快速入门：使用资源管理器创建 Apache Hadoop 群集 - Azure HDInsight
description: 在本快速入门中，将使用资源管理器模板在 Azure HDInsight 中创建 Apache Hadoop 群集
keywords: hadoop 入门,hadoop linux,hadoop 快速入门,hive 入门,hive 快速入门
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 06/12/2019
ms.date: 11/11/2019
ms.author: v-yiso
ms.openlocfilehash: 4b146d4ba72a7f2af54f3c8ba29f87b2072fe078
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92471085"
---
# <a name="quickstart-create-apache-hadoop-cluster-in-azure-hdinsight-using-arm-template"></a>快速入门：使用 ARM 模板在 Azure HDInsight 中创建 Apache Hadoop 群集

本快速入门将使用 Azure 资源管理器模板（ARM 模板）在 Azure HDInsight 中创建 [Apache Hadoop](./apache-hadoop-introduction.md) 群集。 Hadoop 是原始的开源框架，适用于对群集上的大数据集进行分布式处理和分析。 Hadoop 生态系统包括相关的软件和实用程序，例如 Apache Hive、Apache HBase、Spark、Kafka 等等。

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]
  
目前，HDInsight 附带[七个不同的群集类型](../hdinsight-overview.md#cluster-types-in-hdinsight)。 每个群集类型都支持一组不同的组件。 所有群集类型都支持 Hive。 有关 HDInsight 中受支持组件的列表，请参阅 [HDInsight 提供的 Hadoop 群集版本中有哪些新功能？](../hdinsight-component-versioning.md)  

如果你的环境满足先决条件，并且你熟悉如何使用 ARM 模板，请选择“部署到 Azure”按钮。 Azure 门户中会打开模板。

[![部署到 Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-ssh-password%2Fazuredeploy.json)

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-ssh-password/)。

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
    "clusterType": {
      "type": "string",
      "allowedValues": [
        "hadoop",
        "intractivehive",
        "hbase",
        "storm",
        "spark"
      ],
      "metadata": {
        "description": "The type of the HDInsight cluster to create."
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
        "clusterDefinition": {
          "kind": "[parameters('clusterType')]",
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
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D3_v2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              }
            },
            {
              "name": "workernode",
              "targetInstanceCount": 2,
              "hardwareProfile": {
                "vmSize": "Standard_D3_v2"
              },
              "osProfile": {
                "linuxOperatingSystemProfile": {
                  "username": "[parameters('sshUserName')]",
                  "password": "[parameters('sshPassword')]"
                }
              }
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
   
    <a href="https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-ssh-password%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hadoop-linux-tutorial-get-started/hdi-deploy-to-azure1.png" alt="Deploy to Azure"></a>

1. 输入或选择下列值：

    |properties  |说明  |
    |---------|---------|
    |订阅|从下拉列表中选择用于此群集的 Azure 订阅。|
    |资源组|从下拉列表中选择现有资源组，或选择“新建”。|
    |位置|将使用用于资源组的位置自动填充此值。|
    |群集名称|输入任何全局唯一的名称。 对于此模板，请只使用小写字母和数字。|
    |群集类型 | 选择“hadoop”。 |
    |群集登录用户名|提供用户名，默认值为 **admin** 。|
    |群集登录密码|提供密码。 密码长度不得少于 10 个字符，且至少必须包含一个数字、一个大写字母和一个小写字母、一个非字母数字字符（' " ` 字符除外）。 |
    |SSH 用户名|提供用户名，默认值为 **sshuser**|
    |SSH 密码|提供密码。|

    某些属性已在模板中硬编码。  可以通过模板配置这些值。 有关这些属性的详细说明，请参阅[在 HDInsight 中创建 Apache Hadoop 群集](../hdinsight-hadoop-provision-linux-clusters.md)。

    > [!NOTE]  
    > 提供的值必须唯一，并应遵循命名指南。 模板不会执行验证检查。 如果提供的值已被使用，或不遵循指南，则提交模板后可能会出错。  

    ![在门户上开始使用 Resource Manager 模板 (HDInsight Linux)](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-arm-template-on-portal.png "使用 Azure 门户和资源组管理器模板在 HDInsight 中部署 Hadoop 群集")

1. 查看“条款和条件”。 接着选择“我同意上述条款和条件”，然后选择“购买” 。 你会收到一则通知，说明正在进行部署。 创建群集大约需要 20 分钟时间。

## <a name="review-deployed-resources"></a>查看已部署的资源

创建群集后，你会收到“部署成功”通知，通知中附有“转到资源”链接 。 “资源组”页会列出新的 HDInsight 群集以及与该群集关联的默认存储。 每个群集都有一个 [Azure Blob 存储](../hdinsight-hadoop-use-blob-storage.md)帐户和一个 [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md) 依赖项。 该帐户称为默认存储帐户。 HDInsight 群集及其默认存储帐户必须共存于同一个 Azure 区域中。 删除群集不会删除存储帐户。

> [!NOTE]  
> 如需其他群集创建方法或要了解本快速入门中使用的属性，请参阅[创建 HDInsight 群集](../hdinsight-hadoop-provision-linux-clusters.md)。

## <a name="clean-up-resources"></a>清理资源
完成本快速入门后，可以删除群集。 有了 HDInsight，便可以将数据存储在 Azure 存储中，因此可以在群集不用时安全地删除群集。 此外，还需要支付 HDInsight 群集费用，即使未使用。 由于群集费用高于存储空间费用数倍，因此在不使用群集时将其删除可以节省费用。

> [!NOTE]
> 如果立即进行下一教程，了解如何使用 Hadoop on HDInsight 运行 ETL 操作，建议保持群集运行。 这是因为该教程中必须再次创建 Hadoop 群集。 但是，如果不立即学习下一教程，则必须立即删除该群集。

从 Azure 门户导航到群集，然后选择“删除”。

![HDInsight 从门户中删除群集](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-delete-cluster.png "HDInsight 从门户中删除群集")

还可以选择资源组名称来打开“资源组”页，然后选择“删除资源组”。 通过删除资源组，可以删除 HDInsight 群集和默认存储帐户。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已了解了如何使用资源管理器模板在 HDInsight 中创建 Apache Hadoop 群集。 下一篇文章将介绍如何使用 Hadoop on HDInsight 执行提取、转换和加载 (ETL) 操作。

> [!div class="nextstepaction"]
>[使用 HDInsight 上的交互式查询提取、转换和加载数据](../interactive-query/interactive-query-tutorial-analyze-flight-data.md)

[1]: ../HDInsight/apache-hadoop-visual-studio-tools-get-started.md

[hdinsight-provision]: hdinsight-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md
