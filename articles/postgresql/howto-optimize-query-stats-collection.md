---
title: 优化查询统计信息集合 - Azure Database for PostgreSQL（单一服务器）
description: 本文介绍了如何在 Azure Database for PostgreSQL - 单一服务器上优化查询统计信息集合。
author: WenJason
ms.author: v-jay
ms.service: postgresql
ms.topic: how-to
origin.date: 5/6/2019
ms.date: 10/19/2020
ms.openlocfilehash: 17575928691c4af45154ff2962279a3cafcb60db
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041766"
---
# <a name="optimize-query-statistics-collection-on-an-azure-database-for-postgresql---single-server"></a>在 Azure Database for PostgreSQL - 单一服务器上优化查询统计信息集合
本文介绍如何在 Azure Database for PostgreSQL 服务器上优化查询统计信息集合。

## <a name="use-pg_stats_statements"></a>使用 pg_stats_statements
Pg_stat_statements 是 PostgreSQL 扩展，默认情况下在 Azure Database for PostgreSQL 中启用  。 该扩展提供了一种方法来跟踪服务器执行的所有 SQL 语句的执行统计信息。 此模块会挂接到每个查询执行，并且性能成本较高。 启用 **pg_stat_statements** 会强制将查询文本写入到磁盘上的文件。

如果拥有含长查询文本的唯一查询，或者未主动监视 pg_stat_statements，请禁用 pg_stat_statements，以提供最佳性能   。 为此，请将设置更改为 `pg_stat_statements.track = NONE`。

禁用 pg_stat_statements 时，部分工作负荷的性能可实现高达 50% 的提升  。 禁用 pg_stat_statements 的代价是无法对性能问题进行故障排除。

若要设置 `pg_stat_statements.track = NONE`，请执行以下操作：

- 在 Azure 门户中，转到 [PostgreSQL 资源管理页面并选择服务器参数边栏选项卡](howto-configure-server-parameters-using-portal.md)。

  :::image type="content" source="./media/howto-optimize-query-stats-collection/pg_stats_statements_portal.png" alt-text="PostgreSQL 服务器参数边栏选项卡":::

- 使用 [Azure CLI](howto-configure-server-parameters-using-cli.md) az postgres server configuration set `--name pg_stat_statements.track --resource-group myresourcegroup --server mydemoserver --value NONE`。

## <a name="use-the-query-store"></a>使用查询存储 
Azure Database for PostgreSQL 中的[查询存储](concepts-query-store.md)功能提供了用于跟踪查询统计信息的更高效的方法。 建议使用此功能作为使用 pg_stats_statements 的替代方法  。 

## <a name="next-steps"></a>后续步骤
请考虑在 [Azure 门户](howto-configure-server-parameters-using-portal.md)中或通过 [Azure CLI](howto-configure-server-parameters-using-cli.md) 来设置 `pg_stat_statements.track = NONE`。

有关详细信息，请参阅： 
- [Query Store 使用方案](concepts-query-store-scenarios.md) 
- [查询存储最佳做法](concepts-query-store-best-practices.md) 
