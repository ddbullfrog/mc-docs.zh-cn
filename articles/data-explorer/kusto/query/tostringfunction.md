---
title: tostring() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 tostring()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: 3f9dc3f0f93771cbdbf79c65f2ec12aaf48c39eb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105349"
---
# <a name="tostring"></a>tostring()

将输入转换为字符串表示形式。

```kusto
tostring(123) == "123"
```

## <a name="syntax"></a>语法

`tostring(`*`Expr`*`)`

## <a name="arguments"></a>参数

* *`Expr`* ：将转换为字符串的表达式。 

## <a name="returns"></a>返回

如果 `Expr` 值不为 null，则结果将为 `Expr` 的字符串表示形式。
如果 *`Expr`* 值为 null，则结果将为空字符串。
 