---
title: current_cluster_endpoint() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 current_cluster_endpoint()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
ms.openlocfilehash: e3361e6a39aad92b6fc983b777044bc06a71726c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104213"
---
# <a name="current_cluster_endpoint"></a>current_cluster_endpoint()

返回所查询当前群集的网络终结点（DNS 名称）。

## <a name="syntax"></a>语法

`current_cluster_endpoint()`

## <a name="returns"></a>返回

所查询当前群集的网络终结点（DNS 名称），`string` 类型的值。

## <a name="example"></a>示例

```kusto
print strcat("This query executed on: ", current_cluster_endpoint())
```