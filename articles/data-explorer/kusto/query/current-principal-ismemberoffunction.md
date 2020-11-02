---
title: current_principal_is_member_of() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 current_principal_is_member_of()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 03/09/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 6fc6f2225b9098a19f0b39d75bc12263a80c1e7f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104447"
---
# <a name="current_principal_is_member_of"></a>current_principal_is_member_of()

::: zone pivot="azuredataexplorer"

检查运行查询的当前主体的组成员身份或主体标识。

```kusto
print current_principal_is_member_of(
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    )
```

## <a name="syntax"></a>语法

`current_principal_is_member_of`(`*list of string literals*`)

## <a name="arguments"></a>参数

* list of expressions - 以逗号分隔的字符串文本列表，其中每个文本都是一个主体完全限定名称 (FQN) 字符串，其格式为：  
*PrinciplaType*`=`*PrincipalId*`;`*TenantId*

| PrincipalType   | FQN 前缀  |
|-----------------|-------------|
| AAD 用户        | `aaduser=`  |
| AAD 组       | `aadgroup=` |
| AAD 应用程序 | `aadapp=`   |

## <a name="returns"></a>返回
  
该函数返回：
* `true`：如果当前运行查询的主体已成功匹配至少一个输入参数。
* `false`：如果当前运行查询的主体不是任何 `aadgroup=` FQN 参数的成员，并且不等于 `aaduser=` 或 `aadapp=` FQN 参数中的任何一个。
* `(null)`：如果当前运行查询的主体不是任何 `aadgroup=` FQN 参数的成员，并且不等于 `aaduser=` 或 `aadapp=` FQN 参数中的任何一个，且至少有一个 FQN 参数未成功解析（在 Azure AD 中未按下）。 

> [!NOTE]
> 因为该函数返回三态值（`true`、`false` 和 `null`），所以务必仅依赖于正返回值来确认成员身份是否成功。 换言之，以下表达式是不一样的：
> 
> * `where current_principal_is_member_of('non-existing-group')`
> * `where current_principal_is_member_of('non-existing-group') != false` 


## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print result=current_principal_is_member_of(
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    )
```

| result |
|--------|
| (null) |

使用动态数组而不是多个参数：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print result=current_principal_is_member_of(
    dynamic([
    'aaduser=user1@fabrikam.com', 
    'aadgroup=group1@fabrikam.com',
    'aadapp=66ad1332-3a94-4a69-9fa2-17732f093664;72f988bf-86f1-41af-91ab-2d7cd011db47'
    ]))
```

| result |
|--------|
| (null) |

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
