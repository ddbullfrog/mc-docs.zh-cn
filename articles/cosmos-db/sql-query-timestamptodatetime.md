---
title: Azure Cosmos DB 查询语言中的 TimestampToDateTime
description: 了解 Azure Cosmos DB 中的 SQL 系统函数 TimestampToDateTime。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 08/18/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: 0b179f661806d421086b0c657d22276d1b90ee19
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246951"
---
<!--Verified successfully for only charactors-->
# <a name="timestamptodatetime-azure-cosmos-db"></a>TimestampToDateTime (Azure Cosmos DB)

将指定的时间戳值转换为日期/时间。

## <a name="syntax"></a>语法

```sql
TimestampToDateTime (<Timestamp>)
```

## <a name="arguments"></a>参数

*Timestamp*  

一个有符号的数值，即自 Unix 纪元以来当前已经过的毫秒数。 换句话说，返回自 1970 年 1 月 1 日星期四 00:00:00 以来已经过的毫秒数。

## <a name="return-types"></a>返回类型

以 `YYYY-MM-DDThh:mm:ss.fffffffZ` 格式返回 UTC 日期和时间 ISO 8601 字符串值，其中：

  |格式|描述|
  |-|-|
  |YYYY|四位数的年份|
  |MM|两位数的月份（01 = 1 月，依此类推。）|
  |DD|两位数的月份日期（01 到 31）|
  |T|时间元素开头的符号|
  |hh|两位数的小时（00 到 23）|
  |MM|两位数的分钟（00 到 59）|
  |ss|两位数的秒（00 到 59）|
  |.fffffff|七位数的小数秒|
  |Z|UTC（协调世界时）指示符||

<!--Not Available on [!VIDEO https://www.youtube.com/embed/CgYQo6uHyt0]-->

## <a name="remarks"></a>备注

如果指定的时间戳值无效，TimestampToDateTime 将返回 `undefined`。

## <a name="examples"></a>示例

以下示例将时间戳转换为日期/时间：

```sql
SELECT TimestampToDateTime(1594227912345) AS DateTime
```

```json
[
    {
        "DateTime": "2020-07-08T17:05:12.3450000Z"
    }
]
```  

## <a name="next-steps"></a>后续步骤

- [日期和时间函数 Azure Cosmos DB](sql-query-date-time-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: new article about sql query timestamptodatetime -->
<!--NEW.date: 09/28/2020-->