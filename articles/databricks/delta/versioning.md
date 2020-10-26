---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/03/2020
title: 表版本控制 - Azure Databricks
description: 了解如何对 Delta 表进行版本控制。
ms.openlocfilehash: f2bd28c90052508c17c6c21657e5b4a58e01631d
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121813"
---
# <a name="table-versioning"></a><a id="table-version"> </a><a id="table-versioning"> </a>表版本控制

Delta 表的事务日志包含支持 Delta Lake 演变的版本控制信息。 Delta Lake 分别跟踪最小[读取器和写入器版本](delta-utility.md#detail-schema)。

Delta Lake 会保证后向兼容性。 较高版本的 Databricks Runtime 始终能够读取由较低版本写入的数据。

Delta Lake 偶尔会破坏前向兼容性。 较低版本的 Databricks Runtime 可能无法读取和写入由较高版本的 Databricks Runtime 写入的数据。 如果尝试使用太低的 Databricks Runtime 版本对表进行读取和写入操作，则会出现一条错误消息，提示你需要升级。

创建表时，Delta Lake 会根据表的特征（如架构或表属性）选择所需的最低协议版本。 也可以通过设置 SQL 配置来设置默认协议版本：

* `spark.databricks.delta.protocol.minWriterVersion = 2`（默认值）
* `spark.databricks.delta.protocol.minReaderVersion = 1`（默认值）

若要将表升级到更新的协议版本，请使用 `DeltaTable.upgradeTableProtocol` 方法：

## <a name="python"></a>Python

```python
from delta.tables import DeltaTable
delta = DeltaTable.forPath(spark, "path_to_table") # or DeltaTable.forName
delta.upgradeTableProtocol(1, 3) # upgrades to readerVersion=1, writerVersion=3
```

## <a name="scala"></a>Scala

```scala
import io.delta.tables.DeltaTable
val delta = DeltaTable.forPath(spark, "path_to_table") // or DeltaTable.forName
delta.upgradeTableProtocol(1, 3) // upgrades to readerVersion=1, writerVersion=3
```

> [!IMPORTANT]
>
> 协议升级不可逆，因此我们建议你仅在需要时（例如需要选择 Delta Lake 中的新功能时）才升级特定表。