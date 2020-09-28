---
title: 如何删除 HDInsight 群集 - Azure | Azure
description: 删除 HDInsight 群集的各种方式的相关信息。
services: hdinsight
documentationcenter: ''
author: jasonwhowell
manager: cgronlun
editor: cgronlun
ms.assetid: 55f7838b-9786-47ff-96db-1b64437bd0bb
ms.service: hdinsight
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
origin.date: 04/17/2018
ms.date: 10/21/2019
ms.author: v-yiso
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.openlocfilehash: b6e43ed00b3962afe3a88a0cd28036cfad9dbffd
ms.sourcegitcommit: 1118dd532a865ae25a63cf3e7e2eec2d7bf18acc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91394550"
---
# <a name="delete-an-hdinsight-cluster-using-your-browser-powershell-or-the-azure-cli"></a>使用浏览器、PowerShell 或 Azure CLI 删除 HDInsight 群集

创建群集后便开始 HDInsight 群集计费，删除群集后停止计费。 群集以每分钟按比例收费，因此无需再使用群集时，应始终将其删除。 本文档介绍如何使用 Azure 门户、Azure PowerShell 和 Azure CLI 删除群集。

> [!IMPORTANT]
> 删除 HDInsight 群集时不会删除与群集关联的 Azure 存储帐户。 可重新使用以后存储在这些服务中的数据。

## <a name="azure-portal"></a>Azure 门户

1. 登录 [Azure 门户](https://portal.azure.cn)。

2. 在左侧菜单中，导航到“所有服务” > “分析” > “HDInsight 群集”，然后选择群集。

3. 在默认视图中，选择“删除”  图标。 按提示删除群集。
   
    ![删除图标](./media/hdinsight-delete-cluster/hdinsight-delete-cluster.png)

## <a name="azure-powershell"></a>Azure PowerShell

在下面的代码中，将 `CLUSTERNAME` 替换为 HDInsight 群集的名称。 在 PowerShell 提示符处输入以下命令，以便删除群集：

```powershell
Remove-AzHDInsightCluster -ClusterName CLUSTERNAME
```

## <a name="azure-cli"></a>Azure CLI

在下面的代码中，将 `CLUSTERNAME` 替换为 HDInsight 群集的名称，将 `RESOURCEGROUP` 替换为资源组的名称。  在命令提示符处输入以下命令，以便删除群集：

```azurecli
az hdinsight delete --name CLUSTERNAME --resource-group RESOURCEGROUP
```
