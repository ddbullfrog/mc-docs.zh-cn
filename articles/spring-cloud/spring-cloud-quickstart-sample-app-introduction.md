---
title: 快速入门 - 示例应用简介 - Azure Spring Cloud
description: 介绍本系列快速入门中用于部署到 Azure Spring Cloud 的示例应用。
author: MikeDodaro
ms.author: v-junlch
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 11/02/2020
ms.custom: devx-track-java
ms.openlocfilehash: b96b74d7a1838904b0889a4da312e529f6189c48
ms.sourcegitcommit: b9d0091aebc506af49b7cfcd44593711df0d04a5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2020
ms.locfileid: "94373974"
---
# <a name="introduction-to-the-sample-app"></a>示例应用简介

在本快速入门中，我们使用名为 PiggyMetrics 的个人财务示例演示如何将应用部署到 Azure Spring Cloud 服务。 PiggyMetrics 演示了微服务体系结构模式并突出显示了服务明细。 你将了解如何通过强大的 Azure Spring Cloud 功能（包括服务发现、Config Server、日志、指标和分布式跟踪）将其部署到 Azure。

若要遵循 Azure Spring Cloud 部署示例，只需使用根据需要提供的源代码位置。

## <a name="functional-services"></a>功能服务

PiggyMetrics 可分解为三个核心微服务。 所有微服务都是由业务域组织的可独立部署的应用程序。

* **帐户服务（待部署）** ：包含常规用户输入逻辑和验证：收入/支出项、节省额和帐户设置。
* **统计信息服务（本快速入门中未使用）** ：对主要统计信息参数执行计算，并捕获每个帐户的时序。 数据点包含规范化为基本货币和时间段的值。 此数据用于跟踪帐户生存期内的现金流动态。
* **通知服务（本快速入门中未使用）** ：存储用户联系人信息和通知设置，如提醒和备份频率。 计划的辅助进程从其他服务收集所需的信息，并将电子邮件消息发送给订阅的客户。

## <a name="infrastructure-services"></a>基础结构服务

分布式系统中有几种常用模式，有助于核心服务的运行。 Azure Spring Cloud 提供了强大的工具，可增强 Spring Boot 应用程序的行为以实现这些模式： 

* **配置服务（由 Azure Spring Cloud 托管）** ：Azure Spring Cloud Config 是一种面向分布式系统的可水平缩放的集中式配置服务。 它使用当前支持本地存储、Git 和 Subversion 的可插入存储库。
* **服务发现（由 Azure Spring Cloud 托管）** ：它允许自动检测服务实例的网络位置，由于自动缩放、故障和升级，可能会有动态分配的地址。
* **授权服务（待部署）** 授权责任完全提取到单独的服务器，该服务器为后端资源服务授予 OAuth2 令牌。 Auth Server 在外围中进行用户授权和安全的机器对机器通信。
* **API 网关（待部署）** ：这三个核心服务向客户端公开外部 API。 在实际系统中，函数的数量可能会随系统的复杂性迅速增长。 呈现一个复杂的网页可能会涉及数百个服务。 API 网关是系统中的单个入口点，用于处理请求并将请求路由到相应的后端服务，或调用多个后端服务来聚合结果。 

## <a name="sample-usage-of-piggymetrics"></a>PiggyMetrics 的示例用法

有关完整的实现详细信息，请参阅 [PiggyMetrics](https://github.com/Azure-Samples/piggymetrics)。 示例根据需要引用源代码。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [预配 Azure Spring Cloud 实例](spring-cloud-quickstart-provision-service-instance.md)

