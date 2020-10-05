---
title: 规划 Azure 文件部署 | Microsoft Docs
description: 了解规划 Azure 文件部署。 可以直接装载 Azure 文件共享，也可以使用 Azure 文件同步在本地缓存 Azure 文件共享。
author: WenJason
ms.service: storage
ms.topic: conceptual
origin.date: 1/3/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.subservice: files
ms.custom: references_regions
ms.openlocfilehash: 4d0126a3b4fcc4d5dc7203aace82d0ca011ae178
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026530"
---
# <a name="planning-for-an-azure-files-deployment"></a>规划 Azure 文件部署
可通过以下主要方式部署 [Azure 文件存储](storage-files-introduction.md)：直接装载无服务器 Azure 文件共享。

**直接装载 Azure 文件共享**：由于 Azure 文件存储提供 SMB 访问，因此你可以使用 Windows、macOS 和 Linux 中提供的标准 SMB 客户端在本地或云中装载 Azure 文件共享。 由于 Azure 文件共享是无服务器的，因此针对生产方案进行部署不需要管理文件服务器或 NAS 设备。 这意味着，无需应用软件修补程序或交换物理磁盘。 

本文主要阐述有关部署可供本地或云客户端直接装载的 Azure 文件共享时的部署注意事项。

## <a name="management-concepts"></a>管理概念
[!INCLUDE [storage-files-file-share-management-concepts](../../../includes/storage-files-file-share-management-concepts.md)]

将 Azure 文件共享部署到存储帐户时，我们建议：

- 仅将 Azure 文件共享部署到已包含其他 Azure 文件共享的存储帐户。 尽管 GPv2 存储帐户允许使用混合用途的存储帐户，但由于 Azure 文件共享和 Blob 容器等存储资源共享存储帐户的限制，因此，将资源混合在一起可能会导致将来更难以排查性能问题。 

- 部署 Azure 文件共享时请注意存储帐户的 IOPS 限制。 理想情况下，应该以 1:1 的形式将文件共享与存储帐户相映射，但由于组织和 Azure 施加的各种限制和制约，不一定总能实现这种映射。 无法在一个存储帐户中部署一个文件共享时，请考虑哪些共享高度活跃，哪些共享不够活跃，以确保不会将访问量最大的文件共享一起放到同一存储帐户中。

- 仅部署 GPv2 帐户和 FileStorage 帐户，在环境中找到 GPv1 帐户和经典存储帐户时对其进行升级。 

## <a name="identity"></a>标识
若要访问某个 Azure 文件共享，该文件共享的用户必须完成身份验证，并已获得授权。 这种授权是根据访问文件共享的用户的标识完成的。 使用 Azure 存储帐户密钥装载 Azure 文件共享。 若要以这种方式装载文件共享，需使用存储帐户名称作为用户名，使用存储帐户密钥作为密码。 使用存储帐户密钥装载 Azure 文件共享实际上是一项管理员操作，因为装载的文件共享对其上的所有文件和文件夹拥有完全权限，即使对这些文件和文件夹应用了 ACL。 使用存储帐户密钥通过 SMB 装载时，将使用 NTLMv2 身份验证协议。

我们建议根据[网络](#networking)部分中所述使用服务终结点。

## <a name="networking"></a>网络
可在任意位置通过存储帐户的公共终结点访问 Azure 文件共享。 这意味着，已经过身份验证的请求（例如已由用户登录标识授权的请求）可以安全地从 Azure 内部或外部发起。 在许多客户环境中，最初在本地工作站上装载 Azure 文件共享的操作会失败，尽管可以成功地从 Azure VM 装载。 其原因是，许多组织和 Internet 服务提供商 (ISP) 阻止 SMB 用来通信的端口 445。 如需大致了解允许或禁止从端口 445 进行访问的 ISP，请访问 [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx)。

若要取消阻止对 Azure 文件共享的访问，可以采用两种做法：

- 对组织的本地网络取消阻止端口 445。 在外部，只能使用 Internet 安全协议（例如 SMB 3.0 和 FileREST API）通过公共终结点访问 Azure 文件共享。 这是从本地访问 Azure 文件共享的最简单方法，因为它不需要进行高级网络配置，而只需更改组织的出站端口规则；但是，我们建议删除传统的已弃用版本的 SMB 协议，即 SMB 1.0。 若要了解如何执行此操作，请参阅[保护 Windows/Windows Server](storage-how-to-use-files-windows.md#securing-windowswindows-server)和[保护 Linux](storage-how-to-use-files-linux.md#securing-linux)。

- 通过 ExpressRoute 或 VPN 连接访问 Azure 文件共享。 通过网络隧道访问 Azure 文件共享时，可以像装载本地文件共享一样装载 Azure 文件共享，因为 SMB 流量不会通过组织边界。   

尽管从技术角度讲，通过公共终结点装载 Azure 文件共享要容易得多，但我们预期大多数客户会选择通过 ExpressRoute 或 VPN 连接装载其 Azure 文件共享。 为此，需要为环境配置以下设置：  

- **使用 ExpressRoute、站点到站点或点到站点 VPN 的网络隧道**：通过隧道连接到虚拟网络后，即使端口 445 已被阻止，也能从本地访问 Azure 文件共享。
- **DNS 转发**：配置本地 DNS，以将存储帐户的名称（例如 `storageaccount.file.core.chinacloudapi.cn`）解析为专用终结点的 IP 地址。

若要规划与 Azure 文件共享部署相关的网络，请参阅 [Azure 文件存储网络注意事项](storage-files-networking-overview.md)。

## <a name="encryption"></a>Encryption
Azure 文件存储支持两种不同类型的加密：传输中加密（与装载/访问 Azure 文件共享时使用的加密相关），以及静态加密（与存储在磁盘中的数据的加密方式相关）。 

### <a name="encryption-in-transit"></a>传输中加密
默认情况下，所有 Azure 存储帐户均已启用传输中加密。 即通过 SMB 装载文件共享或通过 FileREST 协议（例如，通过 Azure门户、PowerShell/CLI 或 Azure SDK）访问文件共享时，Azure 文件存储仅允许通过加密或 HTTPS 使用 SMB 3.0 及更高版本建立的连接。 如果启用了传输中加密，则不支持 SMB 3.0 的客户端或支持 SMB 3.0 但不支持 SMB 加密的客户端将无法装载 Azure 文件共享。 要详细了解哪些操作系统支持具有加密功能的 SMB 3.0，请参阅适用于 [Windows](storage-how-to-use-files-windows.md)、[macOS](storage-how-to-use-files-mac.md) 和 [Linux](storage-how-to-use-files-linux.md) 的详细文档。 PowerShell、CLI 和 SDK 的所有当前版本均支持 HTTPS。  

可以为 Azure 存储帐户禁用传输中加密。 禁用加密后，Azure 文件存储还将允许没有加密功能的 SMB 2.1、SMB 3.0，以及通过 HTTP 进行的未加密 FileREST API 调用。 禁用传输中加密的主要原因是为了支持必须在更低版本的操作系统（例如，Windows Server 2008 R2 或更低版本的 Linux 发行版）上运行的旧版应用程序。 Azure 文件存储仅允许在与 Azure 文件共享相同的 Azure 区域内建立 SMB 2.1 连接；Azure 文件共享的 Azure 区域之外的 SMB 2.1 客户端（例如，本地或其他 Azure 区域）将无法访问文件共享。

强烈建议启用传输中数据加密。

有关传输中加密的详细信息，请参阅[要求在 Azure 存储中进行安全传输](../common/storage-require-secure-transfer.md?toc=%2fstorage%2ffiles%2ftoc.json)。

### <a name="encryption-at-rest"></a>静态加密
[!INCLUDE [storage-files-encryption-at-rest](../../../includes/storage-files-encryption-at-rest.md)]

## <a name="data-protection"></a>数据保护
Azure 文件有一种多层方法来确保数据得到备份、恢复以及不受安全威胁。

### <a name="soft-delete"></a>软删除
Azure 文件共享的软删除（预览版）是一种存储帐户级别设置，使你在意外删除文件共享时对其进行恢复。 已删除的文件共享会过渡到软删除状态，而非被永久擦除。 可配置软删除数据被永久删除前的可恢复时间，并在此保留期内随时取消删除共享。 

我们建议对大多数文件共享启用软删除。 如果你的工作流中共享删除是常见且预期的，那么你可能会决定很短的保留期，或者根本不启用软删除。

有关软删除的详细信息，请参见[防止意外数据删除](/storage/files/storage-files-prevent-file-share-deletion)。

### <a name="backup"></a>备份
可以通过[共享快照](/storage/files/storage-snapshots-files)备份 Azure 文件共享，这些快照是共享的只读时间点副本。 快照是增量的，这意味着它们只包含自上一个快照以来更改的数据量。 每个文件共享最多可以有 200 个快照，并将其保留长达 10 年。 可以通过 PowerShell 或命令行界面 (CLI) 在 Azure 门户中手动获取这些快照。 快照存储在文件共享中，这意味着如果删除文件共享，快照也将删除。 若要保护快照备份不被意外删除，请确保为共享启用软删除。

## <a name="storage-tiers"></a>存储层
[!INCLUDE [storage-files-tiers-overview](../../../includes/storage-files-tiers-overview.md)]

通常，Azure 文件存储功能以及与其他服务的互操作性在高级文件共享和标准文件共享（包括事务优化文件共享、热文件共享和冷文件共享）之间是相同的，但有几个重要区别：
- **冗余选项**
    - 高级文件共享仅适用于本地冗余 (LRS) 存储。
    - 标准文件共享可用于本地冗余、异地冗余 (GRS) 和异地区域冗余 (GZRS) 存储。
- **文件共享的最大大小**
    - 高级文件共享最多可预配 100 TiB，无需任何额外的操作。
    - 默认情况下，标准文件共享的上限是 5 TiB，但可以通过选择“大文件共享”存储帐户功能标志将共享限制增加到 100 TiB。 对于本地冗余存储帐户或区域冗余存储帐户，标准文件共享的上限是 100 TiB。 有关增加文件共享大小的详细信息，请参阅[启用和创建大文件共享](/storage/files/storage-files-how-to-create-large-file-share)。
- **区域可用性**
    - 高级文件共享并非在每个区域中都可用。 
    - 标准文件共享在每个 Azure 区域中可用。
- Azure Kubernetes 服务 (AKS) 在 1.13 及更高版本中支持高级文件共享。

将文件共享创建为高级文件共享或标准文件共享后，便无法自动将其转换为其他层。 若要切换到其他层，必须在该层中创建新的文件共享，并手动将原始共享中的数据复制到所创建的新共享。 建议使用 `robocopy`（适用于 Windows）或 `rsync`（适用于 macOS 和 Linux）来执行该复制。

### <a name="understanding-provisioning-for-premium-file-shares"></a>了解高级文件共享的预配
高级文件共享是基于固定的 GiB/IOPS/吞吐量比率预配的。 对于预配的每个 GiB，将向该共享分配 1 IOPS 和 0.1 MiB/秒的吞吐量，最多可达每个共享的最大限制。 允许的最小预配为 100 GiB 以及最小的 IOPS/吞吐量。

最大限度地提供服务时，对于预配的存储，所有共享都可以突增到每 GiB 3 IOPS，持续 60 分钟或更长时间，具体取决于共享大小。 新共享将根据预配的容量以完全突增额度开始。

必须以 1 GiB 为增量预配共享。 最小大小为 100 GiB，下一大小为 101 GiB，依此类推。

> [!TIP]
> 基线 IOPS = 1 * 预配的 GiB。 （最大可为 100,000 IOPS）。
>
> 突发限制 = 3 * 基准 IOPS。 （最大可为 100,000 IOPS）。
>
> 出口速率 = 60 MiB/秒 + 0.06 * 预配的 GiB
>
> 入口速率 = 40 MiB/秒 + 0.04 * 预配的 GiB

预配的共享大小按共享配额指定。 随时可以提高共享配额，但只能在自上次提高后的 24 小时之后降低配额。 等待 24 小时且不要提高配额，然后，可将共享配额降低任意次数，直到再次提高配额为止。 IOPS/吞吐量规模更改将在大小更改后的数分钟内生效。

可将预配共享的大小减至所用 GiB 以下。 这样做不会丢失数据，但仍会根据所用大小计费，并且性能（基线 IOPS、吞吐量和突发 IOPS）与预配的共享（而不是所用大小）相符。

下表演示了这些预配共享大小公式的几个示例：

|容量 (GiB) | 基线 IOPS | 突发 IOPS | 出口速率（MiB/秒） | 入口速率（MiB/秒） |
|---------|---------|---------|---------|---------|
|100         | 100     | 最大 300     | 66   | 44   |
|500         | 500     | 最大 1,500   | 90   | 60   |
|1,024       | 1,024   | 最大 3,072   | 122   | 81   |
|5,120       | 5,120   | 最大 15,360  | 368   | 245   |
|10,240      | 10,240  | 最大 30,720  | 675 | 450   |
|33,792      | 33,792  | 最大 100,000 | 2,088 | 1,392   |
|51,200      | 51,200  | 最大 100,000 | 3,132 | 2,088   |
|102,400     | 100,000 | 最大 100,000 | 6,204 | 4,136   |

> [!NOTE]
> 文件共享性能与计算机网络限制、可用网络带宽、IO 大小、并行度和其他许多因素相关。 若要实现最大性能规模，请将负载分散到多个 VM。 有关一些常见性能问题和解决方法，请参阅[故障排除指南](storage-troubleshooting-files-performance.md)。

#### <a name="bursting"></a>突发
高级文件共享最大可按系数 3 突发其 IOPS。 突发是自动进行的，根据额度系统运行。 突发采用“尽力而为”的原则，突发限制没有保证，文件共享只能在限制范围内突发。

每当文件共享的流量低于基线 IOPS 时，额度将累积在突发桶中。 例如，100 GiB 共享有 100 个基线 IOPS。 如果共享中的实际流量在特定 1 秒间隔内为 40 IOPS，则会将 60 个未使用的 IOPS 贷记到突发桶。 以后在操作超过基线 IOPS 时，将使用这些额度。

> [!TIP]
> 突发桶的大小 = 基线 IOPS * 2 * 3600。

每当共享超过基线 IOPS 并且在突发桶中具有额度，就会突发。 只要有剩余的额度，共享就可继续突发，不过，小于 50 TiB 的共享最多只能有一小时不会超过突发限制。 大于 50 TiB 的共享在技术上可以超过此一小时限制（最长可达到两小时），但这取决于应计的突发额度数。 超出基线 IOPS 的每个 IO 会消耗一个额度，一旦耗尽所有额度，共享就会恢复为基线 IOPS。

共享额度具有三种状态：

- 应计：文件共享使用的 IOPS 小于基线 IOPS。
- 下降：文件共享正在突发。
- 保持平稳：没有额度，或正在使用基线 IOPS。

新文件共享最初在其突发桶中包含所有额度。 如果由于服务器的限制，导致共享 IOPS 低于基线 IOPS，则不会对突发额度进行应计。

### <a name="enable-standard-file-shares-to-span-up-to-100-tib"></a>启用标准文件共享最高可以扩展到 100 TiB
[!INCLUDE [storage-files-tiers-enable-large-shares](../../../includes/storage-files-tiers-enable-large-shares.md)]

#### <a name="limitations"></a>限制
[!INCLUDE [storage-files-tiers-large-file-share-availability](../../../includes/storage-files-tiers-large-file-share-availability.md)]

## <a name="redundancy"></a>冗余
[!INCLUDE [storage-files-redundancy-overview](../../../includes/storage-files-redundancy-overview.md)]

## <a name="migration"></a>迁移
在很多情况下，你不想要为组织建立全新的文件共享，而是将现有文件共享从本地文件服务器或 NAS 设备迁移到 Azure 文件存储。 为你的场景选择正确的迁移策略和工具对于迁移的成功非常重要。 

## <a name="next-steps"></a>后续步骤
* [部署 Azure 文件](storage-files-deployment-guide.md)
