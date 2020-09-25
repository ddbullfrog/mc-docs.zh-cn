---
title: 管理中心
description: 在 Azure 数据工厂管理中心管理连接、源代码管理配置和全局创作属性
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: WenJason
ms.author: v-jay
manager: digimobile
origin.date: 06/02/2020
ms.date: 09/21/2020
ms.openlocfilehash: 8cbb964d86708a976816d41870c069d48517edb5
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523729"
---
# <a name="management-hub-in-azure-data-factory"></a>Azure 数据工厂中的管理中心

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

通过 Azure 数据工厂 UX 中的“管理”选项卡访问的管理中心是一个门户，托管数据工厂的全局管理操作。 在这里，你可以管理与数据存储和外部计算的连接以及触发器设置。

## <a name="manage-connections"></a>管理连接

### <a name="linked-services"></a>链接服务

链接服务定义 Azure 数据工厂与外部数据存储和计算环境的连接信息。 有关详细信息，请参阅[链接服务概念](concepts-linked-services.md)。 链接服务的创建、编辑和删除在管理中心完成。

![管理链接服务](media/author-management-hub/management-hub-linked-services.png)

### <a name="integration-runtimes"></a>集成运行时

集成运行时是 Azure 数据工厂用于在不同的网络环境之间提供数据集成功能的计算基础结构。 有关详细信息，请参阅[集成运行时概念](concepts-integration-runtime.md)。 可以在管理中心创建、删除和监视集成运行时。

![管理集成运行时](media/author-management-hub/management-hub-integration-runtime.png)

## <a name="manage-authoring"></a>管理创作

### <a name="triggers"></a>触发器

触发器确定何时应开始管道运行。 目前，触发器可以按挂钟计划触发，可以定期运行，也可以按事件触发。 有关详细信息，请参阅[触发器执行](concepts-pipeline-execution-triggers.md#trigger-execution)。 可以在管理中心创建、编辑、删除触发器或查看触发器的当前状态。

![管理自定义参数](media/author-management-hub/management-hub-triggers.png)

### <a name="global-parameters"></a>全局参数

全局参数是整个数据工厂的常量，可由管道在任何表达式中使用。 有关详细信息，请参阅[全局参数](author-global-parameters.md)。

![创建全局参数](media/author-global-parameters/create-global-parameter-3.png)

