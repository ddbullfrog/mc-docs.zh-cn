---
title: 只读副本 - Azure Database for MySQL
description: 了解 Azure Database for MySQL 中的只读副本：选择区域、创建副本、连接副本、监视复制和停止复制。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: conceptual
origin.date: 10/15/2020
ms.date: 10/29/2020
ms.openlocfilehash: e991d293bd55748140574f80f76cc5d1cc70c5ad
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470434"
---
# <a name="read-replicas-in-azure-database-for-mysql"></a>Azure Database for MySQL 中的只读副本

使用只读副本功能可将数据从 Azure Database for MySQL 服务器复制到只读服务器。 可将源服务器中的数据复制到最多 5 个副本。 使用 MySQL 引擎的基于本机二进制日志 (binlog) 文件位置的复制技术以异步方式更新副本。 若要了解有关 binlog 复制的详细信息，请参阅 [MySQL binlog 复制概述](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html)。

> [!NOTE] 
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

副本是新的服务器，可以像管理普通的 Azure Database for MySQL 服务器一样对其进行管理。 每个只读副本按照预配计算资源的 vCore 数量以及每月 GB 存储量计费。

如需了解有关 MySQL 复制功能和问题的详细信息，请参阅 [MySQL 复制文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html)。

## <a name="when-to-use-a-read-replica"></a>何时使用只读副本

只读副本功能可帮助改善读取密集型工作负荷的性能与规模。 读取工作负载可以与副本服务器隔离，而写入工作负载可以定向到主服务器。

常见方案是让 BI 和分析工作负载将只读副本用作报告的数据源。

由于副本是只读的，它们不能直接缓解主服务器上的写入容量负担。 此功能并非面向写入密集型工作负荷。

只读副本功能使用 MySQL 本机异步复制。 该功能不适用于同步复制方案。 源与副本之间将会存在明显的延迟。 副本上的数据最终将与主服务器上的数据保持一致。 对于能够适应这种延迟的工作负荷，可以使用此功能。

> [!IMPORTANT]
> Azure Database for MySQL 使用基于 ROW 的二进制日志记录。 如果表缺少主键，则会扫描表中的所有行来进行 DML 操作。 这会导致复制延迟时间增加。 若要确保副本服务器能够与源服务器中的更改保持同步，我们通常建议在创建副本服务器之前在源服务器中的表上添加主键，或者在已有副本服务器时重新创建副本服务器。

## <a name="cross-region-replication"></a>跨区域复制
可以在与源服务器不同的区域中创建只读副本。 跨区域复制对于灾难恢复规划或使数据更接近用户等方案非常有用。

任何 [Azure Database for MySQL 区域](https://azure.microsoft.com/global-infrastructure/services/?regions=china-non-regional,china-east,china-east-2,china-north,china-north-2&products=mysql)中都可以有源服务器。  源服务器可以在其配对区域中有副本。

### <a name="paired-regions"></a>配对区域
可以在源服务器的 Azure 配对区域中创建只读副本。 如果你不知道所在区域的配对，可以从 [Azure 配对区域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)一文中了解更多信息。

如果你使用跨区域副本进行灾难恢复规划，建议你在配对区域而不是其他某个区域中创建副本。 配对区域可避免同时更新，并优先考虑物理隔离和数据驻留。  

## <a name="create-a-replica"></a>创建副本

> [!IMPORTANT]
> 只读副本功能仅适用于“常规用途”或“内存优化”定价层中的 Azure Database for MySQL 服务器。 请确保源服务器位于其中一个定价层中。

如果源服务器没有现有的副本服务器，该源服务器会先重启，以便为复制做好自身准备。

启动“创建副本”工作流时，将创建空白的 Azure Database for MySQL 服务器。 新服务器中会填充源服务器上的数据。 创建时间取决于源服务器上的数据量，以及自上次每周完整备份以来所经历的时间。 具体所需时间从几分钟到几小时不等。 始终会在与源服务器相同的资源组和订阅中创建副本服务器。 如果要将副本服务器创建到不同的资源组或不同的订阅，可以在创建后[移动副本服务器](/azure-resource-manager/management/move-resource-group-and-subscription)。

每个副本都启用了存储[自动增长](concepts-pricing-tiers.md#storage-auto-grow)。 自动增长功能允许副本与复制到它的数据保持同步，并防止由于存储空间不足错误而导致的复制中断。

了解如何[在 Azure 门户中创建只读副本](howto-read-replicas-portal.md)。

## <a name="connect-to-a-replica"></a>连接到副本

创建时，副本会继承源服务器的防火墙规则。 之后，这些规则将独立于源服务器。

副本从源服务器继承其管理员帐户。 源服务器上的所有用户帐户都会复制到只读副本。 只能通过使用源服务器上可用的用户帐户来连接到只读副本。

可以使用主机名和有效的用户帐户连接到副本，就像在常规的 Azure Database for MySQL 服务器上连接一样。 对于名称为 **myreplica** 、管理员用户名为 **myadmin** 的服务器，可以使用 mysql CLI 连接到副本：

```bash
mysql -h myreplica.mysql.database.chinacloudapi.cn -u myadmin@myreplica -p
```

在提示符下，输入用户帐户的密码。

## <a name="monitor-replication"></a>监视复制

Azure Database for MySQL 在 Azure Monitor 中提供“复制滞后时间(秒)”指标。 此指标仅适用于副本。 此指标是使用 MySQL 的 `SHOW SLAVE STATUS` 命令中提供的 `seconds_behind_master` 指标计算的。 请设置警报，以便在复制滞后时间达到工作负荷不可接受的值时收到通知。

如果发现复制滞后时间增加，请参阅[排查复制延迟问题](howto-troubleshoot-replication-latency.md)，以便排除故障并了解可能的原因。

## <a name="stop-replication"></a>停止复制

可以停止源与副本之间的复制。 在源服务器与只读副本之间停止复制后，该副本会成为独立服务器。 独立服务器中的数据是启动“停止复制”命令时副本上可用的数据。 独立服务器不会与源服务器保持同步。

选择停止复制到副本时，它会丢失指向其以前的源和其他副本的所有链接。 在源与其副本之间无法自动进行故障转移。

> [!IMPORTANT]
> 独立服务器不能再次成为副本。
> 在只读副本上停止复制之前，请确保副本包含所需的全部数据。

了解如何[停止复制到副本](howto-read-replicas-portal.md)。

## <a name="failover"></a>故障转移

在源服务器和副本服务器之间无法自动进行故障转移。 

由于复制是异步的，因此在源和副本之间存在滞后时间。 滞后时间量会受到多种因素影响，例如，在源服务器上运行的工作负荷有多大，以及数据中心之间的延迟有多严重。 大多数情况下，副本验证在几秒钟到几分钟之间。 可以使用“副本延迟”指标来跟踪实际的副本延迟，该指标适用于每个副本。 该指标显示的是自上次重播事务以来所经历的时间。 建议观察一段时间的副本延迟，以便确定平均延迟。 可以针对副本延迟设置警报，这样，当它超出预期范围时，你就可以采取行动。

> [!Tip]
> 如果你故障转移到副本，在取消副本与源的链接时的滞后时间将表明会丢失多少数据。

一旦决定要故障转移到某个副本， 

1. 请停止将数据复制到副本<br/>
   此步骤是使副本服务器能够接受写入所必需的。 在此过程中，副本服务器会取消与主体的链接。 启动停止复制的操作后，后端进程通常需要大约 2 分钟才能完成。 请参阅本文的[停止复制](#stop-replication)部分，了解此操作的潜在影响。
    
2. 将应用程序指向（以前的）副本<br/>
   每个服务器都有唯一的连接字符串。 更新应用程序，使之指向（以前的）副本而不是主体。
    
如果应用程序成功处理了读取和写入操作，则表明故障转移已完成。 应用程序经历的停机时间取决于何时检测到问题并完成上面的步骤 1 和 2。

## <a name="considerations-and-limitations"></a>注意事项和限制

### <a name="pricing-tiers"></a>定价层

只读副本当前仅适用于“常规用途”和“内存优化”的定价层。

> [!NOTE]
> 运行副本服务器的成本取决于副本服务器的运行区域。

### <a name="source-server-restart"></a>源服务器重启

在为没有现有副本的源创建副本时，该源服务器会先重启，以便为复制做好自身准备。 请考虑这一点并在非高峰期执行这些操作。

### <a name="new-replicas"></a>新副本

只读副本创建为新的 Azure Database for MySQL 服务器。 无法将现有的服务器设为副本。 无法创建另一个只读副本的副本。

### <a name="replica-configuration"></a>副本配置

副本是使用与主服务器相同的服务器配置创建的。 在创建副本后，可以独立于源服务器更改多项设置：计算代系、vCore 数、存储，以及备份保持期。 定价层也可以独立更改，但“基本”层除外。

> [!IMPORTANT]
> 将源服务器的配置更新为新值之前，请将副本配置更新为与这些新值相等或更大的值。 此操作可确保副本与主服务器发生的任何更改保持同步。

在创建副本时，防火墙规则和参数设置会从源服务器继承到副本。 之后，副本的规则便会独立。

### <a name="stopped-replicas"></a>停止的副本

如果停止源服务器与只读副本之间的复制，已停止的副本会成为接受读取和写入操作的独立服务器。 独立服务器不能再次成为副本。

### <a name="deleted-source-and-standalone-servers"></a>已删除的源服务器和独立服务器

在删除源服务器时，会对所有只读副本都停止复制。 这些副本会自动成为独立服务器，并且可以接受读取和写入。 源服务器本身会被删除。

### <a name="user-accounts"></a>用户帐户

源服务器上的用户会复制到只读副本。 只能使用源服务器上可用的用户帐户来连接到只读副本。

### <a name="server-parameters"></a>服务器参数

为了防止数据不同步并避免潜在的数据丢失或损坏，使用读取副本时，会锁定某些服务器参数以防止其更新。

源服务器和副本服务器上都会锁定以下服务器参数：
- [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-multiple-tablespaces.html) 
- [`log_bin_trust_function_creators`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators)

副本服务器上的 [`event_scheduler`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_event_scheduler) 参数处于锁定状态。 

若要在源服务器上更新上述某个参数，请删除副本服务器，更新主服务器上的参数值，然后重新创建副本。

### <a name="other"></a>其他

- 不支持创建副本服务器的副本。
- 内存中的表可能会导致副本服务器变得不同步。这是 MySQL 复制技术的限制。 有关详细信息，请阅读 [MySQL 参考文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features-memory.html)中的更多信息。
- 请确保源服务器表具有主键。 缺少主键可能会导致源和副本之间出现复制延迟。
- 查看 [MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html)中 MySQL 复制限制的完整列表

## <a name="next-steps"></a>后续步骤

- 了解如何[使用 Azure 门户创建和管理只读副本](howto-read-replicas-portal.md)
- 了解如何[通过 Azure CLI 和 REST API 创建和管理只读副本](howto-read-replicas-cli.md)
