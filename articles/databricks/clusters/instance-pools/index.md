---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 池 - Azure Databricks
description: 了解 Azure Databricks 池。
ms.openlocfilehash: 478f88e7418d9ece7b44b49a1573aed7e96a9acf
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121857"
---
# <a name="pools"></a>池

Azure Databricks 池通过维护一组空闲的随时可用的实例来减少群集启动和自动缩放时间。 如果[附加到池](cluster-instance-pool.md#cluster-instance-pool)的群集需要一个实例，它首先会尝试分配池中的一个空闲实例。 如果该池中没有空闲实例，则该池会通过从实例提供程序分配新的实例进行扩展，以满足群集的请求。 当群集释放了某个实例时，该实例将返回到池中，以供其他群集使用。 只有附加到池的群集才能使用该池的空闲实例。

当实例在池中处于空闲状态时，Azure Databricks 不会收取 DBU 费用， 但这会产生实例提供程序费用，具体请参阅[定价](https://www.azure.cn/pricing/details/virtual-machines/linux/)。

可以使用 UI、CLI 或通过调用池 API 来管理池。 此部分介绍如何通过 UI 来使用池。 有关其他方法，请参阅[实例池 CLI](../../dev-tools/cli/instance-pools-cli.md) 和[实例池 API](../../dev-tools/api/latest/instance-pools.md)。

本部分内容：

* [显示池](display.md)
* [创建池](create.md)
* [池配置](configure.md)
* [编辑池](edit.md)
* [删除池](delete.md)
* [使用池](cluster-instance-pool.md)