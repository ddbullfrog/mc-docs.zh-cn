---
title: todatetime() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 todatetime()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 09/30/2020
ms.openlocfilehash: ec761c6452fc9574d0a6e6453b184449a28055a6
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103983"
---
# <a name="todatetime"></a>todatetime()

将输入转换为[日期/时间](./scalar-data-types/datetime.md)标量。

```kusto
todatetime("2015-12-24") == datetime(2015-12-24)
```

## <a name="syntax"></a>语法

`todatetime(`Expr`)`

## <a name="arguments"></a>参数

* Expr：将转换为[日期/时间](./scalar-data-types/datetime.md)的表达式。

## <a name="returns"></a>返回

如果转换成功，则结果将为[日期/时间](./scalar-data-types/datetime.md)值。
否则，结果将为 null。
 
> [!NOTE]
> 尽可能首选使用 [datetime()](./scalar-data-types/datetime.md)。
