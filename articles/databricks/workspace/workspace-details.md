---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 10/05/2020
title: 获取工作区、群集、笔记本、模型和作业标识符 - Azure Databricks
description: 了解如何在 Azure Databricks 中获取工作区实例名称和 ID、群集 URL、笔记本 URL、模型 ID 和作业 URL。
ms.openlocfilehash: 36a33aadcfe3be1a730e86c00668e4c062dae33b
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937771"
---
# <a name="get-workspace-cluster-notebook-model-and-job-identifiers"></a>获取工作区、群集、笔记本、模型和作业标识符

本文将介绍如何在 Azure Databricks 中获取工作区、群集、模型、笔记本、作业标识符和作业 URL。

## <a name="workspace-instance-names-urls-and-ids"></a><a id="workspace-instance-names-urls-and-ids"> </a><a id="workspace-url"> </a>工作区实例名称、URL 和 ID

实例名称会分配给每个 Azure Databricks 部署，并由你登录 Azure Databricks 部署的 URL 中的完全限定域名表示。

Azure Databricks [工作区](index.md) 是运行 Azure Databricks 平台的位置，可在其中创建 Spark 群集和计划工作负载。 工作区具有唯一的数字工作区 ID。

在 Azure 中，每个工作区都分配有两个 URL（每工作区和旧区域），并且获取实例名称和工作区 ID 的方式取决于 URL。

### <a name="per-workspace-url"></a>每工作区 URL

此唯一的每工作区 URL 采用以下格式：`adb-<workspace-id>.<random-number>.databricks.azure.cn`。 工作区 ID 紧跟在 `adb-` 的后面，在圆点 (.) 的前面。 对于每工作区 URL `https://adb-5555555555555555.19.databricks.azure.cn/`：

* 实例名称为 `adb-5555555555555555.19.databricks.azure.cn`。
* 工作区 ID 为 `5555555555555555`。

#### <a name="determine-per-workspace-url"></a>确定每工作区 URL

可确定工作区的每工作区 URL：

* 登录时在浏览器中：

  > [!div class="mx-imgBorder"]
  > ![工作区](../_static/images/workspace/workspace-azure.png)

* 在 Azure 门户中，方式是选择资源，并记下 URL 字段中的值：

  > [!div class="mx-imgBorder"]
  > ![工作区 URL](../_static/images/workspace/azure-workspace-url.png)

### <a name="legacy-regional-url"></a><a id="legacy-regional-url"> </a><a id="legacy-url"> </a>旧区域 URL

> [!IMPORTANT]
>
> 新工作区不支持旧 URL。

旧区域 URL 由部署 Azure Databricks 工作区的区域和域 `databricks.azure.cn`（例如 `https://chinaeast2.databricks.azure.cn/`）组成。

* 如果登录到类似 `https://chinaeast2.databricks.azure.cn/` 的旧区域 URL，则实例名称为 `chinaeast2.databricks.azure.cn`。
* 仅在使用旧区域 URL 登录之后，此 URL 中才会显示工作区 ID。 它显示在 `o=` 的后面。 在 URL `https://<databricks-instance>/?o=6280049833385130` 中，工作区 ID 为 `6280049833385130`。

## <a name="cluster-url-and-id"></a>群集 URL 和 ID

Azure Databricks [群集](../clusters/index.md)为运行生产 ETL 管道、流分析、临时分析和机器学习等各种用例提供了统一平台。 每个群集都有一个被称作群集 ID 的唯一 ID。 这既适用于通用群集，也适用于作业群集。 若要使用 REST API 获取群集的详细信息，必须使用群集 ID。

若要获取群集 ID，请单击边栏中的“群集”选项卡，然后选择群集名称。 群集 ID 是此页面的 URL 中 `/clusters/` 组件后面的数字

`https://<databricks-instance>/#/settings/clusters/<cluster-id>`

在以下屏幕截图中，群集 ID 为：`0831-211914-clean632`。

> [!div class="mx-imgBorder"]
> ![群集 URL](../_static/images/workspace/azure-cluster.png)

## <a name="notebook-url-and-id"></a><a id="notebook-url-and-id"> </a><a id="workspace-notebook-url"> </a>笔记本 URL 和 ID

[笔记本](../notebooks/index.md)是文档的基于 Web 的接口，其中包含可运行的代码、可视化效果和叙述性文本。 笔记本是用于与 Azure Databricks 进行交互的接口。 每个笔记本都具有唯一的 ID。 笔记本 URL 具有笔记本 ID，因此笔记本 URL 对于笔记本而言是唯一的。 可与 Azure Databricks 平台上有权查看和编辑笔记本的任何人共享笔记本 ID。 此外，每个笔记本命令（单元）都有不同的 URL。

若要访问笔记本 URL，请打开笔记本。

在以下笔记本中，笔记本 URL 是 `https://chinaeast2.databricks.azure.cn/?o=6280049833385130#notebook/1940481404050342`，笔记本 ID 是 `1940481404050342`，命令（单元）URL 是

`https://chinaeast2.databricks.azure.cn/?o=6280049833385130#notebook/1940481404050342/command/2432220274659491`

> [!div class="mx-imgBorder"]
> ![笔记本 URL](../_static/images/workspace/azure-notebook.png)

## <a name="model-id"></a>模型 ID

模型指的是 MLflow [已注册的模型](../applications/mlflow/model-registry.md)，你可使用它通过阶段转换和版本控制在生产中管理 MLflow 模型。 通过[权限 API](../dev-tools/api/latest/permissions.md) 以编程方式更改此模型的权限时，需要使用已注册模型的 ID。

若要获取已注册模型的 ID，可使用 [REST API 2.0](../dev-tools/api/latest/index.md) 终结点 `mlflow/databricks/registered-models/get`。 例如，下面的代码会返回已注册模型的对象及其属性，包括其 ID：

```bash
curl -n -X GET -H 'Content-Type: application/json' -d '{"name": "model_name"}' \
https://<databricks-instance>/api/2.0/mlflow/databricks/registered-models/get
```

返回的值采用以下格式：

```json
{
  "registered_model_databricks": {
    "name":"model_name",
    "id":"ceb0477eba94418e973f170e626f4471"
  }
}
```

## <a name="job-url-and-id"></a>作业 URL 和 ID

[作业](../jobs.md)是立即运行或按计划运行笔记本或 JAR 的一种方法。

若要访问作业 URL，请单击边栏中的“作业”选项卡，然后单击作业名称。 若要对失败的作业运行进行故障排除并调查根本原因，必须使用此作业 URL。

在以下屏幕截图中，作业 URL 为 `https://chinaeast2.databricks.azure.cn/?o=6280049833385130#job/1`，作业 ID 为 `1`。

> [!div class="mx-imgBorder"]
> ![作业 URL](../_static/images/workspace/azure-jobs.png)