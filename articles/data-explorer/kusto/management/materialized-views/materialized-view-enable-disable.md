---
title: 启用和禁用具体化视图命令 - Azure 数据资源管理器
description: 本文介绍如何在 Azure 数据资源管理器中启用或禁用具体化视图命令。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: yifats
ms.service: data-explorer
ms.topic: reference
origin.date: 08/30/2020
ms.date: 10/30/2020
ms.openlocfilehash: ab32c481f45b002e61838255281913c713c82b43
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106802"
---
# <a name="disable--enable-materialized-view"></a>.disable | .enable materialized-view

可以通过以下任一方式禁用具体化视图：

* **系统自动禁用：** 如果具体化因永久性错误而失败，则会自动禁用具体化视图。 此过程可能在以下情况下发生： 
    * 与视图定义不一致的架构更改。  
    * 对源表的更改导致具体化视图查询在语义上无效。 
* **显式禁用具体化视图：** 如果具体化视图对群集的运行状况产生负面影响（例如，消耗过多的 CPU），请使用下面的[命令](#syntax)禁用该视图。

> [!NOTE]
> * 禁用具体化视图后，具体化会暂停，不会消耗群集中的资源。 即使具体化视图处于禁用状态，也可以对其进行查询，但性能可能会很差。 已禁用的具体化视图的性能取决于自禁用后引入到源表的记录数。 
> * 可以启用以前已禁用的具体化视图。 具体化视图在重新启用后会继续从中断处具体化，不会跳过任何记录。 如果该视图长时间被禁用，可能需要很长时间才能赶上进度。

仅当你怀疑某个视图影响群集的运行状况时，才建议禁用该视图。

## <a name="syntax"></a>语法

`.enable` | `disable` `materialized-view` *MaterializedViewName*

## <a name="properties"></a>属性

|properties|类型|说明
|----------------|-------|---|
|MaterializedViewName|字符串|具体化视图的名称。|

## <a name="example"></a>示例

```kusto
.enable materialized-view ViewName

.disable materialized-view ViewName
```