---
title: current_principal() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 current_principal()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 12/10/2019
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: eb81cf2352b4528b148905faa93db74a8d7936d8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104446"
---
# <a name="current_principal"></a>current_principal()

::: zone pivot="azuredataexplorer"

返回运行查询的当前主体名称。

## <a name="syntax"></a>语法

`current_principal()`

## <a name="returns"></a>返回

当前主体的完全限定名称 (FQN) 为 `string`。  
字符串格式为：  
PrinciplaType`=`PrincipalId`;`TenantId  

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print fqn=current_principal()
```

|fqn|
|---|
|aaduser=346e950e-4a62-42bf-96f5-4cf4eac3f11e;72f988bf-86f1-41af-91ab-2d7cd011db47|

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
