---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/28/2020
title: MLflow API - Azure Databricks
description: 了解 MLflow REST API。
ms.openlocfilehash: a79d145f0b0188a8bf5bcb1b0e5dd40461715c43
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937673"
---
# <a name="mlflow-api"></a>MLflow API

Azure Databricks 提供了托管版本的 [MLflow](../../../applications/mlflow/index.md) 跟踪服务器和模型注册表，用于托管 [MLflow REST API](https://mlflow.org/docs/latest/rest-api.html)。
可以使用表单的 URL 调用 MLflow REST API

`https://<databricks-instance>/api/2.0/mlflow/<api-endpoint>`

（将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)）。

[MLflow 兼容性矩阵](../../../release-notes/runtime/releases.md#mlflow-compatibility-matrix)列出了每个 Databricks Runtime 版本中打包的 MLflow 版本以及指向相应文档的链接。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="rate-limits"></a>速率限制

根据 MLflow API 的功能和最大吞吐量，其速率限制为四组。
以下是 API 组及其各自限制（以 qps（每秒查询次数）为单位）的列表：

* 低吞吐量试验管理（列出、更新、删除、还原）：7 qps
* 搜索运行：7 qps
* 日志批处理：47 qps
* 所有其他 API：127 qps

此外，每个工作区最多只能有 20 个处于挂起状态（创建中）的并发模型版本。

如果达到此速率限制，后续 API 调用将返回状态代码 429。
所有 MLflow 客户端（包括 UI）都会以指数退避的方式自动重试 429。