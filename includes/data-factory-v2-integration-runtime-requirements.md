---
author: WenJason
ms.service: data-factory
ms.topic: include
origin.date: 07/13/2019
ms.date: 09/21/2020
ms.author: v-jay
ms.openlocfilehash: 5e8c61f25de1f64c55077e746fc5b0237914c444
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523646"
---
<!--
    Separate the generic requirement on Self-hosted Integration Runtime set-up from connector articles.
-->
如果数据存储位于本地网络、Azure 虚拟网络或 Amazon Virtual Private Cloud 内部，则需要设置[自承载集成运行时](../articles/data-factory/create-self-hosted-integration-runtime.md)才能连接到该数据存储。

如果数据存储是托管的云数据服务，则可以使用 Azure 集成运行时。 如果访问范围限制为防火墙规则中列入允许列表的 IP，可以选择将 [Azure 集成运行时 IP](../articles/data-factory/azure-integration-runtime-ip-addresses.md) 添加到允许列表。 

要详细了解网络安全机制和数据工厂支持的选项，请参阅[数据访问策略](../articles/data-factory/data-access-strategies.md)。
