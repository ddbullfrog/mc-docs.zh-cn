---
title: 查询限制策略 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的查询限制策略
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: miwalia
ms.service: data-explorer
ms.topic: reference
origin.date: 10/05/2020
ms.date: 10/30/2020
ms.openlocfilehash: bfde38f2e345b400023da8d561c9a8160f1c7e61
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106782"
---
# <a name="query-throttling-policy"></a>查询限制策略

定义查询限制策略以限制群集同时执行的并发查询的数量。 策略可以在运行时更改，在 alter policy 命令完成后立即进行。

* 使用 [`.show cluster policy querythrottling`](query-throttling-policy-commands.md#show-cluster-policy-querythrottling) 显示群集的当前查询限制策略。
* 使用 [`.alter cluster policy querythrottling`](query-throttling-policy-commands.md#alter-cluster-policy-querythrottling) 设置群集的当前查询限制策略。
* 使用 [`.delete cluster policy querythrottling`](query-throttling-policy-commands.md#delete-cluster-policy-querythrottling) 删除群集的当前查询限制策略。

> [!NOTE]
> 如果未定义查询限制策略，则大量并发查询可能会导致群集无法访问或性能下降。
