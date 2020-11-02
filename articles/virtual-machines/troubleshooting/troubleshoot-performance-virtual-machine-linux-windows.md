---
title: 排查 Linux 或 Windows 上的 Azure 虚拟机性能问题
description: 本文介绍如何通过监视和观察瓶颈来排查一般的虚拟机 (VM) 性能问题，并针对可能发生的问题提供可行的修正救措施。
services: virtual-machines-windows, azure-resource-manager
manager: dcscontentpm
tags: top-support-issue, azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
origin.date: 09/18/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: no
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 8ac9cee1a06709b1f3d1631f65c24c6a582c34fc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104593"
---
# <a name="troubleshoot-azure-virtual-machine-performance-on-linux-or-windows"></a>排查 Linux 或 Windows 上的 Azure 虚拟机性能问题

本文介绍如何通过监视和观察瓶颈来排查一般的虚拟机 (VM) 性能问题，并针对可能发生的问题提供可行的修正救措施。 除了监视之外，还可以使用 Perfinsights。Perfinsights 提供报告，以及最佳做法建议和 IO/CPU/内存方面的关键瓶颈。

<!--Not Available on [Windows](./how-to-use-perfinsights.md)-->
<!--Not Available on [Linux](./how-to-use-perfinsights-linux.md)-->

本文逐步讲解如何使用监视来诊断性能瓶颈。

## <a name="enabling-monitoring"></a>启用监视

### <a name="azure-iaas-virtual-machine-monitoring"></a>Azure IAAS 虚拟机监视

若要监视来宾 VM，请使用 Azure VM 监视，当较高层面的出现某些资源状况时，它可以发出警报。 若要检查是否已启用 VM 诊断，请参阅 [Azure 资源日志概述](../../azure-monitor/learn/tutorial-resource-logs.md)。 如果看到以下消息，则很可能未启用诊断：

:::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/1-virtual-machines-monitoring-not-enabled.png" alt-text="未启用监视":::

### <a name="enable-vm-diagnostics-through-azure-portal"></a>通过 Azure 门户启用 VM 诊断

若要启用 VM 诊断：

1. 转到 VM
2. 单击“诊断设置”
3. 选择存储帐户，然后单击“启用来宾级监视”。

    :::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/2-virtual-machines-diagnostics.png" alt-text="未启用监视":::

可以从“诊断设置”下的“代理”选项卡中检查用于诊断设置的存储帐户。 

:::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/3-check-storage-account.png" alt-text="未启用监视":::

### <a name="enable-storage-account-diagnostics-through-azure-portal"></a>通过 Azure 门户启用存储帐户诊断

当我们打算分析 Azure 中虚拟机的 IO 性能时，存储是一个非常重要的层。 对于与存储相关的指标，需要启用诊断作为一个附加步骤。 如果我们只想分析与存储相关的计数器，也可以启用诊断。

1. 选择 VM 来识别该 VM 使用的存储帐户（或帐户）。 依次单击“设置”、“磁盘”： 

    :::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/4-storage-disks-disks-selection.png" alt-text="未启用监视":::

2. 在门户中，转到 VM 的存储帐户（或帐户）并完成以下步骤：

    1. 单击通过上述步骤找到的存储帐户的概述。
    2. 此时会显示默认指标。 

        :::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/5-default-metrics.png" alt-text="未启用监视":::

3. 单击任一指标，此时会显示另一个边栏选项卡，其中包含用于配置和添加指标的更多选项。

    :::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/6-add-metrics.png" alt-text="未启用监视":::

配置以下选项：

1. 选择“指标”。
2. 选择“资源”（存储帐户）。
3. 选择“命名空间”
4. 选择“指标”。
5. 选择“聚合”的类型
6. 可将此视图固定在仪表板上。

## <a name="observing-bottlenecks"></a>观察瓶颈

完成所需指标的初始设置过程，并为 VM 和相关存储帐户启用诊断后，可以转到分析阶段。

### <a name="accessing-the-monitoring"></a>访问监视功能

选择要调查的 Azure VM，然后选择“监视”。

:::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/7-select-monitoring.png" alt-text="未启用监视":::

### <a name="timelines-of-observation"></a>观察时间线

若要识别是否出现了任何资源瓶颈，请检查数据。 如果你发现计算机运行正常，但报告指出其性能最近已下降，请检查在问题发生期间和之后，涵盖在报告之前性能指标已发生更改的数据的时间范围。

### <a name="check-for-cpu-bottleneck"></a>检查 CPU 瓶颈

:::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/8-cpu-bottleneck-time-range.png" alt-text="未启用监视":::

1. 编辑图形。
2. 设置时间范围。
3. 然后需要添加计数器：来宾 OS CPU 百分比
4. 保存。

### <a name="cpu-observe-trends"></a>观察 CPU 趋势

查看性能问题时，请注意趋势，并了解它们对你产生的影响。 在后续部分，我们将使用门户中的“监视”图形来显示趋势。 还可以使用这些图形来交叉参考同一时间段内的不同资源行为。 若要自定义图形，请单击“[Azure Monitor 数据平台](../../azure-monitor/platform/data-platform.md)”。

出现高峰 - 高峰可能与计划的任务/已知事件相关。 如果可以识别任务，请确定该任务是否以所需的性能级别运行。 如果性能可接受，则可能不需要增加资源。

出现高峰但稳定 - 通常表示出现了新的工作负荷。 如果它不是已识别到的工作负荷，请在 VM 中启用监视，以确定是哪个（或哪些）进程导致了此行为。 识别进程后，确定消耗量的增大是由低效的代码造成的，还是正常的消耗量。 如果消耗量正常，请确定进程是否以所需的性能级别运行。

稳定 - 确定 VM 是始终以此级别运行，还是仅在启用诊断后才以该级别运行。 如果是后者，请确定是哪个（或哪些）进程导致了此问题，并考虑增加该资源。

稳定增大 - 消耗量持续增大通常是低效的代码或者占用更多用户工作负荷的进程造成的。

### <a name="high-cpu-utilization-remediation"></a>修正 CPU 利用率较高的问题

如果应用程序或进程不以适当的性能级别运行，并且你看到 CPU 利用率持续保持在 95% 以上，可执行以下任务之一：

* 立即缓解问题 - 将 VM 的大小增大至提供更多核心的大小
* 查明问题 - 找到相关的应用程序/进程，并相应地进行故障排除。

如果已增大 VM 的大小，但 CPU 利用率仍为 95%，请确定此设置是否能够在可接受的级别提供更好的性能或更高的应用程序吞吐量。 如果不是，请排查单个应用程序/进程的问题。

<!--Not Available on Perfinsights-->
<!--Not Available on [Windows](./how-to-use-perfinsights.md)-->
<!--Not Available on [Linux](./how-to-use-perfinsights-linux.md)-->

## <a name="check-for-memory-bottleneck"></a>检查内存瓶颈

若要查看指标：

1. 添加一个部分。
2. 添加一个磁贴。
3. 打开库。
4. 选择“内存用量”并拖动鼠标。 停靠该磁贴后，单击右键并选择“6x4”。

### <a name="memory-observe-trends"></a>观察内存趋势

“内存用量”将显示 VM 正在消耗的内存量。 了解趋势，以及它是否与出现问题时的时间相对应。 应该始终有 100 MB 以上的可用内存。

出现高峰但稳定/消耗量稳定 - 内存利用率较高不一定是性能不佳的原因，因为某些应用程序（例如关系数据库引擎）会分配大量内存，但内存利用率可能并不很高。 不过，如果有多个应用程序消耗大量的内存，则内存争用可能会造成性能不佳，从而导致修整和分页/交换到磁盘。 这种性能不佳往往是应用程序性能受影响的显著原因。

消耗量稳定增大 - 可能是应用程序正在“预热”，每次启动数据库引擎时，都经常会出现这种消耗模式。 然而，这也可能是应用程序内存泄漏的迹象。 识别该应用程序并了解该行为是否在意料之中。

页面或交换文件用量 - 检查是否在重度使用 Windows 分页文件 (位于 D:\) 或 Linux 交换文件（位于 `/dev/sdb`）。 如果这些卷仅包含这些文件，请检查这些磁盘上的读/写次数是否较高。 此问题表示出现内存不足的情况。

### <a name="high-memory-utilization-remediation"></a>修正内存利用率较高的问题

若要解决内存利用率较高的问题，请执行以下任一任务：

* 立即缓解页面或交换文件的用量问题 - 将 VM 大小增大至具有更多内存的大小，然后监视 VM。
* 查明问题 - 找到相关的应用程序/进程，并进行故障排除，以识别内存消耗量较高的应用程序。
* 如果你知道该应用程序，请确定是否可以限制内存分配。

如果在升级到更大的 VM 后发现内存利用率仍然持续增大，直到 100%，请识别相关的应用程序/进程并进行故障排除。

<!--Not Available on Perfinsights-->
<!--Not Available on [Windows](./how-to-use-perfinsights.md)-->
<!--Not Available on [Linux](./how-to-use-perfinsights-linux.md)-->

## <a name="check-for-disk-bottleneck"></a>检查磁盘瓶颈

若要检查 VM 的存储子系统，请使用 VM 诊断中的计数器以及存储帐户诊断，来检查 Azure VM 级别的诊断数据。

<!--Not Available on Perfinsights-->
<!--Not Available on [Windows](./how-to-use-perfinsights.md)-->
<!--Not Available on [Linux](./how-to-use-perfinsights-linux.md)-->

请注意，我们不提供区域冗余和高级存储帐户的计数器。 如果遇到这些计数器相关的问题，请提出支持案例。

### <a name="viewing-storage-account-diagnostics-in-monitoring"></a>在监视中查看存储帐户诊断数据

若要处理以下各项，请在门户中转到 VM 的存储帐户：

:::image type="content" source="media/troubleshoot-performance-virtual-machine-linux-windows/9-virtual-machine-storage-account.png" alt-text="未启用监视":::

1. 编辑监视图形。
2. 设置时间范围。
3. 添加以下步骤中所述的计数器。
4. 保存更改。

### <a name="disk-observe-trends-standard-storage-only"></a>观察磁盘趋势（仅限标准存储）

若要识别存储的问题，请查看存储帐户诊断和 VM 诊断中的性能指标。

对于以下每项检查，请查看发生问题的时间范围内的关键趋势。

#### <a name="check-azure-storage-availability---add-the-storage-account-metric-availability"></a>检查 Azure 存储可用性 - 添加存储帐户指标：可用性

如果发现可用性下降，则可能表示平台出现了问题，此时请检查 [Azure 状态](https://status.azure.com/status/)。 如果 Azure 状态未显示问题，请提出新的支持请求。

#### <a name="check-for-azure-storage-timeout---add-the-storage-account-metrics"></a>检查 Azure 存储超时 - 添加存储帐户指标：

* ClientTimeOutError
* ServerTimeOutError
* AverageE2ELatency
* AverageServerLatency
* TotalRequests

*TimeOutError 指标中的值指示某个 IO 操作花费了太长时间并已超时。完成后续步骤可帮助识别潜在原因。

如果在发生 TimeOutError 的同时 AverageServerLatency 增大，则可能表示平台出现了问题。 对于这种情况，请提出新的支持请求。

AverageE2ELatency 表示客户端延迟。 验证应用程序如何执行 IOPS。 查找增大的或持续保持高位的 TotalRequests 指标。 此指标表示 IOPS。 如果开始达到存储帐户或单个 VHD 的限制，则延迟可能与限制相关。

#### <a name="check-for-azure-storage-throttling---add-the-storage-account-metrics-throttlingerror"></a>检查 Azure 存储限制 - 添加存储帐户指标：ThrottlingError

限制的值指示在存储帐户级别受到限制，即，达到了帐户的 IOPS 限制。 可以通过检查指标 **TotalRequests** 来确定是否达到了 IOPS 阈值。

请注意，每个 VHD 限制为 500 IOPS 或 60 Mb，但不能超过每个存储帐户 20000 IOPS 的累积限制。

通过此指标无法判断哪个 Blob 导致了限制，以及哪些 Blob 受到了限制的影响。 但是，可以肯定达到了存储帐户的 IOPS 限制或入口/出口限制。

若要识别是否达到了 IOPS 限制，请转到存储帐户诊断，并检查 TotalRequests 来确定 TotalRequest 数目是否接近 20000 个。 识别模式的更改、是否是首次出现限制，或者此限制是否在特定的时间发生。

对于以标准存储提供的新磁盘产品/服务，IOPS 和吞吐量限制可能不同，但标准存储帐户的累积限制为 20000 IOPS（高级存储在帐户或磁盘级别实施不同的限制）。 详细了解不同的标准存储磁盘产品/服务和每个磁盘的限制：

* [Windows 上的 VM 磁盘的可伸缩性和性能目标](../windows/disk-scalability-targets.md)。

#### <a name="references"></a>参考

* [高级页 blob 存储帐户的可伸缩性和性能目标](../../storage/blobs/scalability-targets-premium-page-blobs.md)

存储帐户的带宽按以下存储帐户指标进行度量：TotalIngress 和 TotalEgress。 带宽的阈值根据冗余和区域类型的不同而异。

* [标准存储帐户的可伸缩性和性能目标](../../storage/common/scalability-targets-standard-account.md)

根据存储帐户冗余类型和区域的入口和出口限制检查 TotalIngress 和 TotalEgress。

检查附加到 VM 的 VHD 的吞吐量限制。 添加 VM 磁盘读取和写入指标。

以标准存储提供的新磁盘产品/服务实施不同的 IOPS 和吞吐量限制（IOPS 不会按 VHD 公开）。 查看数据，使用“磁盘读取和写入”来确定是否在 VM 级别达到了 VHD 合并吞吐量 (MB) 限制，然后优化 VM 存储配置来克服单个 VHD 的限制。 详细了解不同的标准存储磁盘产品/服务和每个磁盘的限制：

* [Windows 上的 VM 磁盘的可伸缩性和性能目标](../windows/disk-scalability-targets.md)。

### <a name="high-disk-utilizationlatency-remediation"></a>修正磁盘利用率/延迟较高的问题

降低客户端延迟并优化 VM IO 来克服 VHD 限制

* [优化 Azure 中 Windows 的 IO](../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md?toc=/virtual-machines/windows/toc.json)

* [优化 Azure 中 Linux 的 IO](https://docs.microsoft.com/archive/blogs/igorpag/azure-storage-secrets-and-linux-io-optimizations)

#### <a name="reduce-throttling"></a>降低限制

如果达到了存储帐户的上限，请重新均衡存储帐户之间的 VHD。 请参阅 [Azure 存储可伸缩性和性能目标](../../storage/common/scalability-targets-standard-account.md)。

### <a name="increase-throughput-and-reduce-latency"></a>提高吞吐量并降低延迟

如果你有延迟敏感型应用程序并需要较高的吞吐量，请使用 DS 和 GS 系列 VM 将 VHD 迁移到 Azure 高级存储。

以下文章介绍了具体的方案：

* [迁移到 Azure 高级存储](../windows/migrate-to-managed-disks.md)

* [在 SQL Server 上使用 Azure 高级存储](https://docs.microsoft.com/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-server-premium-storage)

## <a name="next-steps"></a>后续步骤

如果对本文中的任何观点存在疑问，请联系 [Azure 支持](https://support.azure.cn/support/contact/)上的 Azure 专家。

或者，提交 Azure 支持事件。 请转到 [Azure 支持站点](https://support.azure.cn/support/support-azure/)提交请求。

<!-- Update_Description: update meta properties, wording update, update link -->