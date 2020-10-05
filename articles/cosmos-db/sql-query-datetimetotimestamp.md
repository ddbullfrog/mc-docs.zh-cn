---
title: Azure Cosmos DB 查询语言中的 DateTimeToTimestamp
description: 了解 Azure Cosmos DB 中的 SQL 系统函数 DateTimeToTimestamp。
ms.service: cosmos-db
ms.topic: conceptual
origin.date: 08/18/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.custom: query-reference
ms.openlocfilehash: 2251a2c1da1bc7f2ed1bf40d14de751aea408f3a
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246952"
---
<!--Verified Successfully-->
# <a name="datetimetotimestamp-azure-cosmos-db"></a>DateTimeToTimestamp (Azure Cosmos DB)

将指定的日期/时间转换为时间戳。

## <a name="syntax"></a>语法

```sql
DateTimeToTimestamp (<DateTime>)
```

## <a name="arguments"></a>参数

*DateTime*  
   `YYYY-MM-DDThh:mm:ss.fffffffZ` 格式的 UTC 日期和时间 ISO 8601 字符串值，其中：

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

## <a name="return-types"></a>返回类型

返回一个有符号的数值，表示自 Unix 纪元以来当前已经过的毫秒数，即自 1970 年 1 月 1 日星期四 00:00:00 以来已经过的毫秒数。

## <a name="remarks"></a>备注

如果指定的日期/时间值无效，DateTimeToTimestamp 将返回 `undefined`

## <a name="examples"></a>示例

以下示例将日期/时间转换为时间戳：

```sql
SELECT DateTimeToTimestamp("2020-07-09T23:20:13.4575530Z") AS Timestamp
```

```json
[
    {
        "Timestamp": 1594336813457
    }
]
```  

再提供一个示例：

```sql
SELECT DateTimeToTimestamp("2020-07-09") AS Timestamp
```

```json
[
    {
        "Timestamp": 1594252800000
    }
]
```  

## <a name="next-steps"></a>后续步骤

- [日期和时间函数 Azure Cosmos DB](sql-query-date-time-functions.md)
- [系统函数 Azure Cosmos DB](sql-query-system-functions.md)
- [Azure Cosmos DB 简介](introduction.md)

<!-- Update_Description: new article about sql query datetimetotimestamp -->
<!--NEW.date: 09/28/2020-->