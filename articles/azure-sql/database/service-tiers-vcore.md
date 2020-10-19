---
title: vCore 购买模型概述
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: 借助 vCore 购买模型，可单独缩放计算和存储资源、匹配本地性能，并优化 Azure SQL 数据库和 Azure SQL 托管实例的价格。
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.topic: conceptual
author: WenJason
ms.author: v-jay
ms.reviewer: sashan, moslake, carlrab
origin.date: 08/14/2020
ms.date: 09/14/2020
ms.openlocfilehash: 10f4dff2d3d9e055d1487762e36fabb2d6fc5a8b
ms.sourcegitcommit: 1810e40ba56bed24868e573180ae62b9b1e66305
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91872415"
---
# <a name="vcore-model-overview---azure-sql-database-and-azure-sql-managed-instance"></a>vCore 模型概述 - Azure SQL 数据库和 Azure SQL 托管实例 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL 数据库和 Azure SQL 托管实例使用的虚拟核心 (vCore) 购买模型具有下面几个优势：

- 更高的计算、内存、I/O 和存储限制。
- 控制硬件代系，以便更好地符合工作负荷的计算和内存要求。
- [Azure 混合权益 (AHB)](../azure-hybrid-benefit.md) 的定价折扣。
- 为驱动计算的硬件细节提供更高的透明度，这有助于规划从本地部署进行的迁移。

## <a name="service-tiers"></a>服务层

vCore 模型中的服务层级选项包括“常规用途”、“业务关键”和“超大规模”。 服务层级通常定义存储体系结构、空间和 I/O 限制，以及与可用性和灾难恢复相关的业务连续性选项。

|-|**常规用途**|**业务关键**|**超大规模**|
|---|---|---|---|
|最适用于|大多数业务工作负荷。 提供预算导向的、均衡且可缩放的计算和存储选项。 |它使用多个独立副本为商业应用程序提供最高级别的故障恢复能力，为每个数据库副本提供最高的 I/O 性能。|具有很高的可缩放存储和读取缩放要求的大多数业务工作负荷。  允许配置多个独立的数据库副本，提供更高的故障恢复能力。 |
|存储|使用远程存储。<br/>**SQL 数据库预配计算**：<br/>5 GB - 4 TB<br/>**无服务器计算**<br/>5 GB - 3 TB<br/>**SQL 托管实例**：32 GB - 8 TB |使用本地 SSD 存储。<br/>**SQL 数据库预配计算**：<br/>5 GB - 4 TB<br/>**SQL 托管实例**：<br/>32 GB - 2 TB |可以根据需要灵活地自动扩展存储。 最多支持 100 TB 存储空间。 使用本地 SSD 存储作为本地缓冲池缓存和本地数据存储。 使用 Azure 远程存储作为最终的长期数据存储。 |
|IOPS 和吞吐量（近似值）|**SQL 数据库**：请查看[单一数据库](resource-limits-vcore-single-databases.md)和[弹性池](resource-limits-vcore-elastic-pools.md)的资源限制。<br/>**SQL 托管实例**：请参阅 [Azure SQL 托管实例资源限制概述](../managed-instance/resource-limits.md#service-tier-characteristics)。|请查看[单一数据库](resource-limits-vcore-single-databases.md)和[弹性池](resource-limits-vcore-elastic-pools.md)的资源限制。|超大规模是具有多个级别缓存的多层体系结构。 有效的 IOPS 和吞吐量将取决于工作负载。|
|可用性|1 个副本，无读取缩放副本|3 个副本，1 个[读取缩放副本](read-scale-out.md)|1 个读写副本加 0-4 个[读取缩放副本](read-scale-out.md)|
|备份|[读取访问异地冗余存储 (RA-GRS)](../../storage/common/geo-redundant-design.md)，7-35 天（默认为 7 天）|[RA-GRS](../../storage/common/geo-redundant-design.md)，7-35 天（默认为 7 天）|Azure 远程存储中基于快照的备份。 还原使用这些快照进行快速恢复。 备份瞬间完成，不会影响计算 I/O 性能。 还原速度很快，不基于数据操作的大小（需要几分钟，而不是几小时或几天）。|
|内存中|不支持|支持|不支持|
|||


### <a name="choosing-a-service-tier"></a>选择服务层级

有关为特定工作负荷选择服务层级的信息，请参阅以下文章：

- [何时选择“常规用途”服务层级](service-tier-general-purpose.md#when-to-choose-this-service-tier)
- [何时选择“业务关键”服务层级](service-tier-business-critical.md#when-to-choose-this-service-tier)
- [何时选择“超大规模”服务层级](service-tier-hyperscale.md#who-should-consider-the-hyperscale-service-tier)


## <a name="compute-tiers"></a>计算层级

vCore 模型中的计算层级选项包括预配计算层级和无服务器计算层级。


### <a name="provisioned-compute"></a>预配计算

预配计算层级提供特定数量的计算资源，这些资源是独立于工作负荷活动持续预配的，针对预配的计算资源量按固定的小时价格计费。


### <a name="serverless-compute"></a>无服务器计算

[无服务器计算层级](serverless-tier-overview.md)根据工作负荷活动自动缩放计算资源，针对使用的计算资源量按秒计费。



## <a name="hardware-generations"></a>硬件代系

vCore 模型中的硬件代系选项包括“第 4 代”和“第 5 代”。 硬件代系通常定义计算和内存限制，以及影响工作负荷性能的其他特征。

### <a name="gen4gen5"></a>第 4 代/第 5 代

- 第 4 代/第 5 代硬件提供均衡的计算和内存资源，适合用于内存和 vCore 要求不太高，或者不需要较快的单一 vCore 的大部分数据库工作负荷，而 Fsv2 系列或 M 系列可以满足这些要求。

有关第 4 代/第 5 代的可用区域，请参阅[第 4 代/第 5 代可用性](#gen4gen5-1)。

### <a name="compute-and-memory-specifications"></a>计算和内存规格


|硬件代次  |计算  |内存  |
|:---------|:---------|:---------|
|Gen4     |- Intel® E5-2673 v3 (Haswell) 2.4 GHz 处理器<br>- 最多预配 24 个 vCore（1 个 vCore = 1 个物理核心）  |- 每个 vCore 7 GB<br>- 最多预配 168 GB|
|Gen5     |**预配计算**<br>- Intel® E5-2673 v4 (Broadwell) 2.3-GHz、Intel® SP-8160 (Skylake)\* 和 Intel® 8272CL (Cascade Lake) 2.5 GHz\* 处理器<br>- 最多预配 80 个 vCore（1 个 vCore = 1 个超线程）<br><br>**无服务器计算**<br>- Intel® E5-2673 v4 (Broadwell) 2.3-GHz 和 Intel® SP-8160 (Skylake)* 处理器<br>- 自动扩展为 40 个 vCore（1 个 vCore = 1 个超线程）|**预配计算**<br>- 每个 vCore 5.1 GB<br>- 最多预配 408 GB<br><br>**无服务器计算**<br>- 自动扩展为每个vCore 24 GB<br>- 自动纵向扩展为最大 120 GB|

\* 在 [sys.dm_user_db_resource_governance](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database) 动态管理视图中，使用 Intel® SP-8160 (Skylake) 处理器的数据库的硬件代系会显示为 Gen6，而使用 Intel® 8272CL (Cascade Lake) 的数据库的硬件代系会显示为 Gen7。 不管处理器类型如何（Broadwell、Skylake 或 Cascade Lake），所有 Gen5 数据库的资源限制都相同。

有关资源限制的详细信息，请参阅[单一数据库的资源限制 (vCore)](resource-limits-vcore-single-databases.md) 或[弹性池的资源限制 (vCore)](resource-limits-vcore-elastic-pools.md)。

### <a name="selecting-a-hardware-generation"></a>选择硬件代系

在 Azure 门户中，可在 SQL 数据库中创建数据库或池时为其选择硬件代系，或者更改现有 SQL 数据库或池的硬件代系。

**创建 SQL 数据库或池时选择硬件代系**

有关详细信息，请参阅[创建 SQL 数据库](single-database-create-quickstart.md)。

在“基本信息”选项卡上，选择“计算 + 存储”部分中的“配置数据库”链接，然后选择“更改配置”链接：   

  ![配置数据库](./media/service-tiers-vcore/configure-sql-database.png)

选择所需的硬件代系：

  ![选择硬件](./media/service-tiers-vcore/select-hardware.png)


**更改现有 SQL 数据库或池的硬件代系**

对于数据库，请在“概述”页上选择“定价层”链接：

  ![更改硬件](./media/service-tiers-vcore/change-hardware.png)

对于池，请在“概述”页上选择“配置”。

遵循相应的步骤更改配置，然后根据前面的步骤所述选择硬件代系。

**创建 SQL 托管实例时选择硬件代系**

有关详细信息，请参阅[创建 SQL 托管实例](../managed-instance/instance-create-quickstart.md)。

在“基本信息”选项卡上，选择“计算 + 存储”部分中的“配置数据库”链接，然后选择所需的硬件代系：  

  ![配置 SQL 托管实例](./media/service-tiers-vcore/configure-managed-instance.png)
  
**更改现有 SQL 托管实例的硬件代系**

# <a name="the-azure-portal"></a>[Azure 门户](#tab/azure-portal)

在“SQL 托管实例”页面上，选择“设置”部分下的“定价层”链接

![更改 SQL 托管实例硬件](./media/service-tiers-vcore/change-managed-instance-hardware.png)

在“定价层”页上，可以按前面步骤中所述更改硬件代系。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

使用以下 PowerShell 脚本：

```powershell
Set-AzSqlInstance -Name "managedinstance1" -ResourceGroupName "ResourceGroup01" -ComputeGeneration Gen5
```

有关更多详细信息，请查看 [Set-AzSqlInstance](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstance) 命令。

# <a name="the-azure-cli"></a>[Azure CLI](#tab/azure-cli)

使用以下 CLI 命令：

```azurecli
az sql mi update -g mygroup -n myinstance --family Gen5
```

有关更多详细信息，请查看 [az sql mi update](/cli/sql/mi#az-sql-mi-update) 命令。

---

### <a name="hardware-availability"></a>硬件可用性

#### <a name="gen4gen5"></a><a name="gen4gen5-1"></a> 第 4 代/第 5 代

第 5 代可在中国东部 2 和中国北部 2 区域中使用。 第 4 代可在中国东部和中国北部区域中使用。

## <a name="next-steps"></a>后续步骤

如要入门，请参阅： 
- [使用 Azure 门户创建 SQL 数据库](single-database-create-quickstart.md)
- [使用 Azure 门户创建 SQL 托管实例](../managed-instance/instance-create-quickstart.md)

有关定价详细信息，请参阅 [Azure SQL 数据库定价页](https://azure.cn/pricing/details/sql-database/single/)。

若要详细了解“常规用途”和“业务关键”服务层级中提供的特定计算大小和存储大小，请参阅：

- [Azure SQL 数据库基于 vCore 的资源限制](resource-limits-vcore-single-databases.md)。
- [共用 Azure SQL 数据库基于 vCore 的资源限制](resource-limits-vcore-elastic-pools.md)。
- [Azure SQL 托管实例基于 vCore 的资源限制](../managed-instance/resource-limits.md)。 

