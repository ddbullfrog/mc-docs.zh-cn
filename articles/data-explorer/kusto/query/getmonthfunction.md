---
title: getmonth() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 getmonth()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/22/2020
ms.date: 10/29/2020
ms.openlocfilehash: fd36743fd5ee9eda77b6e7ffee74b1ee3e1606c1
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103895"
---
# <a name="getmonth"></a>getmonth()

从日期时间获取月份号 (1-12)。

另一个别名：monthoyear()

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print month = getmonth(datetime(2015-10-12))
```
