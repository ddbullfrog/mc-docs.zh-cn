---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: REST API 2.0 - Azure Databricks
description: 了解 Databricks REST API 2.0 支持的服务。
ms.openlocfilehash: 795cdce9c70dd0e8702bf5dc4cbb3eec4399b7f6
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937678"
---
# <a name="rest-api-20"></a><a id="rest-api-20"> </a><a id="rest-api-v2"> </a>REST API 2.0

Databricks REST API 2.0 支持用于管理工作区、DBFS、群集、实例池、作业、库、用户和组、令牌以及 MLflow 试验和模型的服务。

本文概述了如何使用 REST API。 本文末尾列出了各个 API 参考、身份验证选项和示例的链接。

若要了解如何使用个人访问令牌向 REST API 进行身份验证，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](authentication.md)。 有关 API 示例，请参阅 [API 示例](examples.md)。

若要了解如何使用 Azure Active Directory 令牌向 REST API 进行身份验证，请参阅[使用 Azure Active Directory 令牌进行身份验证](aad/index.md)。 有关示例，请参阅[使用用户的 Azure AD 访问令牌](aad/app-aad-token.md#use-token)和[使用服务主体的 AAD 访问令牌](aad/service-prin-aad-token.md#use-token)。

## <a name="rate-limits"></a>速率限制

Databricks REST API 支持每个工作区最多 30 个请求/秒。 超过速率限制的请求会出现 [429 响应状态代码](https://developer.mozilla.org/docs/Web/HTTP/Status/429)。

## <a name="parse-output"></a>分析输出

分析 JSON 输出的各个部分可能很有用。 在这种情况下，建议使用实用工具 `jq`。 有关详细信息，请参阅 [jq 手册](https://stedolan.github.io/jq/manual/)。 可以通过运行 `brew install jq` 来使用 Homebrew 在 MacOS 上安装 `jq`。

某些 `STRING` 字段（其中包含供 UI 使用的错误/描述性消息）未结构化，在编程工作流中请不要依赖这些字段的格式。

## <a name="invoke-a-get-using-a-query-string"></a>使用查询字符串来调用 GET

尽管大多数 API 调用都要求必须指定 JSON 正文，但对于 `GET` 调用，可以指定查询字符串。

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

若要获取群集的详细信息，请运行：

```bash
curl ... https://<databricks-instance>/api/2.0/clusters/get?cluster_id=<cluster-id>
```

若要列出 DBFS 根目录的内容，请运行：

```bash
curl ... https://<databricks-instance>/api/2.0/dbfs/list?path=/
```

## <a name="runtime-version-strings"></a><a id="programmatic-version"> </a><a id="runtime-version-strings"> </a>Runtime 版本字符串

许多 API 调用都要求必须指定 [Databricks Runtime](../../../runtime/index.md) 版本字符串。 此部分介绍了 Databricks REST API 中的版本字符串的结构。

### <a name="databricks-runtime"></a>Databricks Runtime

```
<M>.<F>.x[-cpu][-gpu][-ml][-hls][conda]-scala<scala-version>
```

其中

* `M` - Databricks Runtime 主要版本
* `F` - Databricks Runtime 功能版
* `cpu` - CPU 版本（只包含 `-ml`）
* `gpu` - [已启用 GPU](../../../clusters/gpu.md#gpu-clusters)
* `ml` - [机器学习](../../../runtime/mlruntime.md#mlruntime)
* `hls` - [基因组学](../../../runtime/genomicsruntime.md#dbr-genomics)
* `conda` - [带有 Conda](../../../runtime/conda.md#condaruntime)（不再可用）
* `scala-version` - 用于编译 Spark 的 Scala 的版本：2.10、2.11 或 2.12

例如，`5.5.x-scala2.10` 和 `6.3.x-gpu-scala2.11`。 [支持的版本](../../../release-notes/runtime/releases.md#supported-list)和[终止支持历史记录](../../../release-notes/runtime/releases.md#unsupported-list)表将 Databricks Runtime 版本映射到运行时中包含的 Spark 版本。

### <a name="databricks-light"></a>Databricks Light

```
apache-spark.<M>.<F>.x-scala<scala-version>
```

其中

* `M` - Apache Spark 主要版本
* `F` - Apache Spark 功能版
* `scala-version` - 用于编译 Spark 的 Scala 的版本：2.10 或 2.11

例如，`apache-spark-2.4.x-scala2.11`。

## <a name="apis"></a>API

* [API 示例](examples.md)
* [使用 Azure Active Directory 令牌进行身份验证](aad/index.md)
* [使用 Azure Databricks 个人访问令牌进行身份验证](authentication.md)
* [群集 API](clusters.md)
* [群集策略 API](policies.md)
* [DBFS API](dbfs.md)
* [组 API](groups.md)
* [实例池 API](instance-pools.md)
* [作业 API](jobs.md)
* [库 API](libraries.md)
* [MLflow API](mlflow.md)
* [权限 API](permissions.md)
* [SCIM API](scim/index.md)
* [机密 API](secrets.md)
* [令牌 API](tokens.md)
* [令牌管理 API](token-management.md)
* [工作区 API](workspace.md)