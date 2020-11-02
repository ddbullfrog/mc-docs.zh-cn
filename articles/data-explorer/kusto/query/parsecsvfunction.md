---
title: parse_csv() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 parse_csv()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: bb2c3cb2fc42bc2e6d1b89577322b900a5aeea78
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104607"
---
# <a name="parse_csv"></a>parse_csv()

拆分表示单个记录（包含逗号分隔值）的给定字符串，并返回包含这些值的字符串数组。

```kusto
parse_csv("aaa,bbb,ccc") == ["aaa","bbb","ccc"]
```

## <a name="syntax"></a>语法

`parse_csv(`*source*`)`

## <a name="arguments"></a>参数

* *source* ：表示单个记录（包含逗号分隔值）的源字符串。

## <a name="returns"></a>返回

一个包含拆分值的字符串数组。

**备注**

可以使用双引号 ('"') 来转义嵌入行的源、逗号和引号。 此函数不支持每行多条记录（仅获取第一条记录）。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print result=parse_csv('aa,"b,b,b",cc,"Escaping quotes: ""Title""","line1\nline2"')
```

|result|
|---|
|[<br>  "aa",<br>  "b,b,b",<br>  "cc",<br>  "Escaping quotes:\"Title\"",<br>  "line1\nline2"<br>]|

具有多条记录的 CSV 有效负载：

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print result_multi_record=parse_csv('record1,a,b,c\nrecord2,x,y,z')
```

|result_multi_record|
|---|
|[<br>  "record1",<br>  "a",<br>  "b",<br>  "c"<br>]|
