---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 06/18/2020
title: Riak 时序 - Azure Databricks
description: 了解如何使用 Azure Databricks 在 Riak TS 时序数据库中读取和写入数据。
ms.openlocfilehash: 4b9d125a6cf253ba14547a1d7b66d61f991a18e2
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121915"
---
# <a name="riak-time-series"></a><a id="riak-time-series"> </a><a id="riak-ts"> </a>Riak 时序

[Riak 时序](https://riak.com/products/riak-ts/index.html)是专为 IoT 和时序数据而优化的企业级 NoSQL 时序数据库。

> [!NOTE]
>
> 无法从运行 Databricks Runtime 7.0 或更高版本的群集访问此数据源，因为支持 Apache Spark 3.0 的 Riak 时序连接器不可用。

下面的笔记本显示了如何开始使用 Riak 时序数据库。

## <a name="riak-time-series-database-notebook"></a>Riak 时序数据库笔记本

[获取笔记本](../../_static/notebooks/riakts.html)