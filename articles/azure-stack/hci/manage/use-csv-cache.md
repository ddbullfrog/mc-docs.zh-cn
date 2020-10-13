---
title: 将 CSV 内存中读取缓存与 Azure Stack HCI 配合使用
description: 本主题介绍如何使用系统内存提高性能。
author: WenJason
ms.author: v-jay
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
origin.date: 09/04/2020
ms.date: 10/12/2020
ms.openlocfilehash: 15ea23d6a4b5aef0877c50f130727909f76b6ddc
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451246"
---
# <a name="use-the-csv-in-memory-read-cache-with-azure-stack-hci"></a>将 CSV 内存中读取缓存与 Azure Stack HCI 配合使用

> 适用于：Azure Stack HCI 版本 20H2；Windows Server 2019

本主题介绍如何使用系统内存提高 Azure Stack HCI 的性能。

Azure Stack HCI 与群集共享卷 (CSV) 内存中读取缓存兼容。 对于使用未缓冲 I/O 访问 VHD 或 VHDX 文件的应用程序（如 Hyper-V），使用系统内存缓存读取可以性能。 （未缓冲 I/O 是未由 Windows 缓存管理器缓存的任何操作。）

由于内存中缓存是服务器本地缓存，因此它可改善数据区域：最近读取会缓存在运行虚拟机的相同主机上的内存中，从而减少通过网络进行读取的频率。 这可实现更低的延迟和更好的存储性能。

请注意，CSV 内存中读取缓存与 Azure Stack HCI 中的[存储池缓存](../concepts/cache.md)不同。

## <a name="planning-considerations"></a>规划注意事项

内存中读取缓存对于读取密集型工作负载（如虚拟桌面基础结构 (VDI)）最有效。 相反，如果工作负载是极端的写入密集型，则缓存引入的开销可能会高于获得的价值，因此应禁用。

可以将多达 80% 的总物理内存用于 CSV 内存中读取缓存。 请注意为虚拟机留出足够的内存！

  > [!NOTE]
  > 某些微基准工具（如 DISKSPD 和 [VM Fleet](https://github.com/Microsoft/diskspd/tree/master/Frameworks/VMFleet)）在启用了 CSV 内存中读取缓存时得到的结果可能比未启用时更差。 默认情况下，VM Fleet 会为每个虚拟机创建一个 10 GiB VHDX（对于 100 个 VM 总共将近 1 TiB），然后对它们执行均匀随机读取和写入。 与实际工作负载不同，读取不遵循任何可预测或重复的模式，因此内存中缓存不起作用，只会产生开销。

## <a name="configuring-the-in-memory-read-cache"></a>配置内存中读取缓存

CSV 内存中读取缓存在具有相同功能的 Azure Stack HCI、Windows Server 2019 和 Windows Server 2016 中可用。 在 Azure Stack HCI 和 Windows Server 2019 中，它在默认情况下处于打开状态，分配有 1 GiB。 在 Windows Server 2016 中，它在默认情况下处于关闭状态。

| OS 版本          | 默认 CSV 缓存大小 |
|---------------------|------------------------|
| Azure Stack HCI     | 1 GiB                  |
| Windows Server 2019 | 1 GiB                  |
| Windows Server 2016 | 0（禁用）           |

若要使用 PowerShell 查看分配的内存量，请运行：

```PowerShell
(Get-Cluster).BlockCacheSize
```

返回的值针对每台服务器，以 MiB 为单位。 例如，`1024` 表示 1 GiB。

若要更改分配的内存量，请使用 PowerShell 修改此值。 例如，若要为每台服务器分配 2 GiB，请运行：

```PowerShell
(Get-Cluster).BlockCacheSize = 2048
```

若要使更改立即生效，请暂停，然后恢复 CSV 卷，或在服务器之间移动它们。 例如，使用此 PowerShell 片段将每个 CSV 移动到另一个服务器节点，并再次返回：

```PowerShell
Get-ClusterSharedVolume | ForEach {
    $Owner = $_.OwnerNode
    $_ | Move-ClusterSharedVolume
    $_ | Move-ClusterSharedVolume -Node $Owner
}
```

## <a name="next-steps"></a>后续步骤

如需相关信息，另请参阅：

- [了解存储池缓存](../concepts/cache.md)
- [在故障转移群集中使用群集共享卷](https://docs.microsoft.com/windows-server/failover-clustering/failover-cluster-csvs#enable-the-csv-cache-for-read-intensive-workloads-optional)
