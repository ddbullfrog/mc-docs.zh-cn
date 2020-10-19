---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/11/2020
title: 为群集启用表访问控制 - Azure Databricks
description: 了解管理员如何为 Azure Databricks 群集启用 Python 和 SQL 表访问控制。
ms.openlocfilehash: c0924434955dc1e5b860bba8565b01dd6cac65c0
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937742"
---
# <a name="enable-table-access-control-for-a-cluster"></a>为群集启用表访问控制

本文介绍如何为群集启用表访问控制。

有关在群集上启用了表访问控制后如何对数据对象设置特权的信息，请参阅[数据对象特权](object-privileges.md)。

## <a name="enable-table-access-control-for-a-cluster"></a><a id="enable-table-access-control-for-a-cluster"> </a><a id="enable-table-acl"> </a>为群集启用表访问控制

表访问控制以两个版本提供：

* [仅限 SQL 的表访问控制](#sql-only-table-access-control)，这是
  * 正式发布版。
  * 将群集用户限制为使用 SQL 命令。 用户限制为只能使用 Apache Spark SQL API，因此无法使用 Python、Scala、R、RDD API 或直接从云存储中读取数据的客户端（例如 DBUtils）。
* [Python 和 SQL 表访问控制](#python-and-sql-table-access-control)，这是
  * 公共预览版。
  * 允许用户运行 SQL、Python 和 PySpark 命令。 用户限制为只能使用 Spark SQL API 和数据帧 API，因此无法使用 Scala、R、RDD API 或直接从云存储中读取数据的客户端（例如 DBUtils）。

### <a name="sql-only-table-access-control"></a><a id="sql-only-table-access-control"> </a><a id="sql-only-table-acl"> </a>仅限 SQL 的表访问控制

此版本的表访问控制将群集上的用户限制为仅使用 SQL 命令。

若要在群集上启用仅限 SQL 表的访问控制并将该群集限制为仅使用 SQL 命令，请在群集的 [Spark 配置](../../../clusters/configure.md#spark-config)中设置以下标志：

```ini
spark.databricks.acl.sqlOnly true
```

> [!NOTE]
>
> 对仅限 SQL 表访问控制的访问权限不受管理员控制台中[启用表访问控制](../../../administration-guide/access-control/table-acl.md)设置的影响。 此设置仅控制是否在工作区范围启用 Python 和 SQL 表访问控制。

### <a name="python-and-sql-table-access-control"></a><a id="python-and-sql-table-access-control"> </a><a id="python-sql-table-acl"> </a>Python 和 SQL 表访问控制

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

此版本的表访问控制允许用户运行使用数据帧 API 和 SQL 的 Python 命令。 在群集上启用此功能后，该群集或池中的用户将：

* 只能通过 Spark SQL API 或数据帧 API 访问 Spark。 在这两种情况下，都根据 Azure Databricks [数据治理模型](object-privileges.md#data-governance-model)由管理员来限制对表和视图的访问。
* 无法通过 DBFS 或通过从云提供商的元数据服务读取凭据来获取对云中数据的直接访问权限。
* 必须在群集节点上运行其命令，因为低特权用户禁止访问文件系统的敏感部分或创建与 80 和 443 以外的端口的网络连接。
  * 只有内置 Spark 函数可以在 80 和 443 以外的端口上创建网络连接。
  * 只有管​​理员用户或具有[任意文件](object-privileges.md#data-governance-model)特权的用户才能通过 [PySpark JDBC 连接器](../../../data/data-sources/sql-databases.md#python-jdbc-example)从外部数据库读取数据。
  * 如果希望 Python 进程能够访问其他出站端口，可以将 [Spark 配置](../../../clusters/configure.md#spark-config) `spark.databricks.pyspark.iptable.outbound.whitelisted.ports` 设置为要加入允许列表的端口。 支持的配置值格式为 `[port[:port][,port[:port]]...]`，例如：`21,22,9000:9999`。  端口必须位于有效范围内，即 `0-65535`。

尝试绕过这些限制将失败，并出现异常。 有这些限制，则你的用户永远无法通过群集访问非特权数据。

#### <a name="requirements"></a>要求

在用户可以配置 Python 和 SQL 表访问控制之前，Azure Databricks 管理员必须：

* 为 Azure Databricks 工作区启用表访问控制。
* 拒绝用户访问未启用表访问控制的群集。 在实践中，这意味着拒绝大多数用户创建群集的权限，并拒绝用户对未为表访问控制启用的群集的“可以附加到”权限。

有关这两种要求的信息，请参阅[为工作区启用表访问控制](../../../administration-guide/access-control/table-acl.md#enable-table-acl)。

#### <a name="create-a-cluster-enabled-for-table-access-control"></a><a id="create-a-cluster-enabled-for-table-access-control"> </a><a id="table-access-control"> </a>创建启用了表访问控制的群集

创建集群时，单击“启用表访问控制并仅允许使用 Python 和 SQL 命令”选项。 此选项仅对于[高并发](../../../clusters/configure.md#high-concurrency)群集可用。

> [!div class="mx-imgBorder"]
> ![启用表访问控制](../../../_static/images/access-control/table-acl-enable-cluster-azure.png)

若要使用 REST API 创建群集，请参阅[有关创建启用了表访问控制的群集的示例](../../../dev-tools/api/latest/examples.md#cluster-table-acl-example)。

## <a name="set-privileges-on-a-data-object"></a>设置对数据对象的特权

请参阅[数据对象特权](object-privileges.md)。