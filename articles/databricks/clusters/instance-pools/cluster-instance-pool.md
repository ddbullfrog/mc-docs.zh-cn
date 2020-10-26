---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 使用池 - Azure Databricks
description: 了解如何在群集中使用池。
ms.openlocfilehash: b5d73d6196968f67f6164ada9b0ac041fea83279
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121792"
---
# <a name="use-a-pool"></a><a id="cluster-instance-pool"> </a><a id="use-a-pool"> </a>使用池

若要缩短群集启动时间，可以将群集附加到包含空闲实例的预定义池。 附加到池后，群集会通过池分配其驱动程序节点和工作器节点。 如果池中没有足够的空闲资源来满足群集的请求，则池会通过从实例提供程序分配新的实例进行扩展。 终止附加的群集后，它使用的实例会返回到池中，可供其他群集重复使用。

## <a name="requirements"></a>要求

你必须有权附加到池；请参阅[池访问控制](../../security/access-control/pool-acl.md)。

## <a name="attach-a-cluster-to-a-pool"></a>将群集附加到池

若要将群集附加到池，请在“池”下拉列表中选择该池。

> [!div class="mx-imgBorder"]
> ![选择池](../../_static/images/clusters/instance-pool.png)

## <a name="inherited-configuration"></a>继承的配置

当群集附加到池后，将从池中继承以下配置属性：

* [群集节点类型](../configure.md#node-types)：不能选择单独的驱动程序和工作器节点类型。
* [自定义群集标记](../configure.md#cluster-tags)：可以为群集添加其他自定义标记，群集级别标记和从池继承的标记都会得到应用。 添加特定于群集的自定义标记时，其键名不能与从池继承的自定义标记的键名相同（也就是说，不能重写从池继承的自定义标记）。