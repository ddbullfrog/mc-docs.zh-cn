---
ms.openlocfilehash: f8f2860e4d42ec181ec51578ba2a30f5c3c2d984
ms.sourcegitcommit: 221c32fe6f618679a63f148da7382bc9e495f747
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92223113"
---
----
title: include 文件 description: include 文件 author: rockboyfor ms.service: virtual-machines ms.topic: include origin.date:09/30/2019 ms.date:07/06/2020 ms.author: v-yeche
---

随着托管在 Azure 中的 VM 数目的显著增长，必须确定影响它们支持的应用程序和基础结构服务的性能和运行状况问题。 默认情况下，会在 Azure 中通过各种指标类型（由主机监控程序收集的 CPU 使用率、磁盘使用率、内存使用率和网络流量）进行基本的监视。 可以使用[扩展](../articles/virtual-machines/windows/extensions-features.md)收集其他指标和日志数据，以便在来宾操作系统的 VM 上配置诊断。

为了检测并诊断 VM 中运行的来宾操作系统、基于 .NET 的 Web 应用程序或 Java Web 应用程序组件的性能和运行状况问题，Azure Monitor 提供了集中监视功能和各种其他功能，例如用于 VM 的 Azure Monitor、Application Insights。

## <a name="diagnostics-and-metrics"></a>诊断和指标 

可以在 Azure 门户、Azure CLI、Azure PowerShell 和编程应用程序编程接口 (API) 中使用[指标](../articles/monitoring-and-diagnostics/monitoring-overview-metrics.md)来设置和监视[诊断数据](https://docs.azure.cn/cli/vm/diagnostics?view=azure-cli-latest)收集。 例如，可以：

- **观察 VM 的基本指标。** Azure 门户的“概述”屏幕上显示的基本指标包括 CPU 使用率、网络使用情况、总磁盘字节数以及每秒的磁盘操作数。

- **使用 Azure 门户启用并查看启动诊断数据收集。** 将自己的映像加载到 Azure 或者启动某个平台映像时，可能会因为许多原因而导致 VM 进入无法启动状态。 创建 VM 时，针对“设置”屏幕的“监视”部分下的“启动诊断”单击“已启用”，即可轻松启用启动诊断。

    VM 启动时，启动诊断代理将捕获启动输出并将其存储在 Azure 存储中。 此数据可以用于排查 VM 启动问题。 从命令行工具创建 VM 时，不会自动启用启动诊断。 在启用启动诊断之前，需要创建一个存储帐户来存储启动日志。 如果在 Azure 门户中启用启动诊断，则会自动创建一个存储帐户。

    如果未在创建 VM 时启用启动诊断，可在以后随时使用 [Azure CLI](https://docs.azure.cn/cli/vm/boot-diagnostics?view=azure-cli-latest)、[Azure PowerShell](https://docs.microsoft.com/powershell/module/az.compute/set-azvmbootdiagnostic) 或 [Azure 资源管理器模板](../articles/virtual-machines/windows/extensions-diagnostics-template.md)启用它。

- **启用来宾 OS 诊断数据收集。** 创建 VM 时，可以在“设置”屏幕上启用来宾 OS 诊断。 如果确实启用了诊断数据收集，[用于 Linux 的 IaaSDiagnostics 扩展](../articles/virtual-machines/linux/diagnostic-extension.md)或[用于 Windows 的 IaaSDiagnostics 扩展](../articles/virtual-machines/windows/ps-extensions-diagnostics.md)将添加到 VM，使你可以收集更多的磁盘、CPU 和内存数据。

    使用收集的诊断数据，可以为 VM 配置自动缩放。 还可以配置 [Azure Monitor 日志](../articles/azure-monitor/platform/data-platform-logs.md)，以便存储数据并设置警报，在性能不正常时通知你。

## <a name="alerts"></a>警报

可以根据特定的性能指标创建[警报](../articles/azure-monitor/platform/alerts-overview.md)。 举例来说，可以根据以下问题生成警报，平均 CPU 使用率超过特定的阈值，或者可用磁盘空间低于特定的空间量。 可以通过 [Azure 门户](../articles/azure-monitor/platform/alerts-metric.md#create-with-azure-portal)、[Azure 资源管理器模板](../articles/azure-monitor/platform/alerts-metric-create-templates.md)或 [Azure CLI](../articles/azure-monitor/platform/alerts-metric.md#with-azure-cli) 配置警报。

## <a name="azure-service-health"></a>Azure 服务运行状况

[Azure 服务运行状况](../articles/service-health/service-health-overview.md)会在 Azure 服务问题影响你时提供个性化的指导和支持，并且会帮助你为即将到来的计划内维护做好准备。 Azure 服务运行状况使用具有针对性和灵活性的通知提醒你和你的团队。

## <a name="azure-resource-health"></a>Azure 资源运行状况

[Azure 资源运行状况](../articles/service-health/resource-health-overview.md)有助于在 Azure 问题影响资源时进行诊断和获取支持。 它通知你有关资源的当前和过去运行状况的信息，并帮助你缓解问题。 在需要有关 Azure 服务问题的帮助时，资源运行状况将提供技术支持。

## <a name="azure-activity-log"></a>Azure 活动日志

[Azure 活动日志](../articles/azure-monitor/platform/platform-logs-overview.md)是一种方便用户深入了解 Azure 中发生的订阅级别事件的订阅日志。 该日志包括从 Azure 资源管理器操作数据到服务运行状况事件更新的一系列数据。 可以在 Azure 门户中单击“活动日志”查看 VM 的日志。

可以对活动日志执行的部分操作包括：

- [根据活动日志事件创建警报](../articles/azure-monitor/platform/platform-logs-overview.md)。
    
    <!--Not Available on - [Stream it to an Event Hub](../articles/azure-monitor/platform/activity-logs-stream-event-hubs.md)-->
    
- 使用 [Power BI 内容包](https://powerbi.microsoft.com/documentation/powerbi-content-pack-azure-audit-logs/)在 Power BI 中进行分析。

    <!--Not Available on - [Save it to a storage account](../articles/azure-monitor/platform/archive-activity-log.md)-->
    
还可以通过使用 [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.insights/)、[Azure CLI](https://docs.azure.cn/cli/monitor?view=azure-cli-latest) 或[监视 REST API](https://docs.microsoft.com/rest/api/monitor/) 访问活动日志数据。

[Azure 资源日志](../articles/azure-monitor/platform/platform-logs-overview.md)是 VM 发出的日志，其中提供与该 VM 的操作相关的各种频繁生成的数据。 不同于活动日志，资源日志提供有关在 VM 中执行的操作的见解。

可以对诊断日志执行的部分操作包括：

- 将诊断日志保存到[存储帐户](../articles/azure-monitor/platform/archive-diagnostic-logs.md)进行审核或手动检查。 可以使用“资源诊断设置”指定保留时间（天）。
    
    <!--Not Available on - [Stream them to Event Hubs](../articles/azure-monitor/platform/resource-logs-stream-event-hubs.md)-->
    
- 使用 [Log Analytics](../articles/log-analytics/log-analytics-azure-storage.md) 对诊断日志进行分析。

## <a name="advanced-monitoring"></a>高级监视

若要查看 Azure VM 和虚拟机规模集支持的应用程序或服务，确定在 VM 中运行的来宾 OS 或工作负载的问题，以便了解它是影响应用程序的可用性或性能，还是应用程序本身出现了问题，请启用 [Application Insights](../articles/azure-monitor/app/app-insights-overview.md)。

<!--Not Available on both [Azure Monitor for VMs](../articles/azure-monitor/insights/vminsights-overview.md) and-->

用于 VM 的 Azure Monitor 分析 Windows 和 Linux VM 的性能与运行状况，包括不同的进程以及与它发现的其他资源和外部进程之间的相互依赖关系，可以大规模监视 Azure 虚拟机 (VM)。 它包含多个趋势性能图表，用于调查问题和评估 VM 的容量。 依赖项映射显示受监视的和不受监视的计算机、进程和这些计算机之间的失败网络连接和有效网络连接，并显示趋势图表，其中包含标准的网络连接指标。 通过组合使用 Application Insights，可以监视应用程序并捕获遥测数据（例如 HTTP 请求、异常等），这样就可以将 VM 和应用程序之间的问题关联起来。 请配置 [Azure Monitor 警报](../articles/azure-monitor/platform/alerts-overview.md)，这样，当系统从用于 VM 的 Azure Monitor 收集的监视数据中检测到重要情况时，就会提醒你。

## <a name="next-steps"></a>后续步骤

- 逐步完成[使用 Azure PowerShell 监视 Windows 虚拟机](../articles/virtual-machines/windows/tutorial-monitoring.md)或[使用 Azure CLI 监视 Linux 虚拟机](../articles/virtual-machines/linux/tutorial-monitoring.md)中的步骤。

<!-- Not Availablle on [Monitoring and diagnostics](https://docs.microsoft.com/azure/architecture/best-practices/monitoring)-->

<!--Update_Description: update meta properties, wording update, update link -->