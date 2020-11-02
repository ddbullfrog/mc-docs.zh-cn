---
title: current_database() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 current_database()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: fe0da55e4d15db0b66345aad93751e9d2fbc2226
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104212"
---
# <a name="current_database"></a>current_database()

返回作用域中数据库的名称（如果未指定其他数据库，则将解析所有查询实体的数据库）。

## <a name="syntax"></a>语法

`current_database()`

## <a name="returns"></a>返回

作用域中数据库的名称作为 `string` 类型的值。

## <a name="example"></a>示例

```kusto
print strcat("Database in scope: ", current_database())
```