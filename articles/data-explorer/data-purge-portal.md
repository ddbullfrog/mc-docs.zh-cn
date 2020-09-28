---
title: 使用数据清除从 Azure 数据资源管理器中的设备或服务中删除个人数据
description: 了解如何使用数据清除从 Azure 数据资源管理器中清除（删除）数据。
author: orspod
ms.author: v-tawe
ms.reviewer: kedamari
ms.service: data-explorer
ms.topic: how-to
origin.date: 05/12/2020
ms.date: 09/24/2020
ms.openlocfilehash: 2ac91ea76ab2fb2db1f9e6fd97d6ddc5d3402fb4
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146450"
---
# <a name="enable-data-purge-on-your-azure-data-explorer-cluster"></a>在 Azure 数据资源管理器群集上启用数据清除

[!INCLUDE [gdpr-intro-sentence](includes/gdpr-intro-sentence.md)]

Azure 数据资源管理器支持删除单个记录。 通过 `.purge` 命令删除数据可保护个人数据，且不应在其他情况下使用。 它不是为支持频繁的删除请求或删除大量数据而设计的，并且可能会对服务的性能产生重大影响。

执行 `.purge` 命令会触发一个可能需要几天时间才能完成的过程。 如果应用了 `predicate` 的记录的“密度”很大，该过程会重新引入表中的所有数据。 此过程会极大影响性能和 COGS。 有关详细信息，请参阅 [Azure 数据资源管理器中的数据清除](kusto/concepts/data-purge.md)。

## <a name="methods-of-invoking-purge-operations"></a>清除操作调用方法 

Azure 数据资源管理器 (Kusto) 支持删除单个记录和清除整个表。 根据使用方案的不同，`.purge` 命令可以[通过两种方式](kusto/concepts/data-purge.md#purge-table-tablename-records-command)调用：

* 编程调用：要由应用程序调用的单个步骤。 调用此命令将直接触发清除执行序列。

* 人为调用：一个两步过程，需要显式确认为单个步骤。 调用该命令将返回验证令牌，需要提供该令牌才能运行实际清除。 此过程降低了无意中删除错误数据的风险。 对于含有重要冷缓存数据的大型表，使用此选项可能需要很长时间才能完成相应过程。 

## <a name="prerequisites"></a>先决条件

* 如果没有 Azure 订阅，请在开始前创建一个[试用 Azure 帐户](https://wd.azure.cn/pricing/1rmb-trial/)。
* 登录到 [Web UI](https://dataexplorer.azure.cn/)。
* 创建 [Azure 数据资源管理器群集和数据库](create-cluster-database-portal.md)

## <a name="enable-data-purge-on-your-cluster"></a>启用群集上的数据清除

> [!WARNING]
> * 启用数据清除需要重启服务，这可能会导致查询删除发生。
> * 启用数据清除之前，请查看[限制](#limitations)。

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集。 
1. 在“设置”中选择“配置”。**** **** 
1. 在“配置”窗格中，选择“打开”以启用“启用清除”**** **** ****。
1. 选择“保存” ****。
 
    ![启用清除](media/data-purge-portal/enable-purge-on.png)

## <a name="disable-data-purge-on-your-cluster"></a>禁用群集上的数据清除

1. 在 Azure 门户中，转到 Azure 数据资源管理器群集。 
1. 在“设置”中选择“配置”。**** **** 
1. 在“配置”窗格中，选择“关闭”以禁用“启用清除”**** **** ****。
1. 选择“保存” ****。

    ![禁用清除](media/data-purge-portal/enable-purge-off.png)

## <a name="limitations"></a>限制

* 清除过程是最终的且不可逆的。 无法“撤消”此过程或恢复已清除的数据。 因此，[undo table drop](kusto/management/undo-drop-table-command.md) 等命令无法恢复清除的数据，将数据回滚到以前的版本也无法恢复到清除“之前”。
* `.purge` 命令对数据管理终结点执行： https://ingest- [YourClusterName].[Region].kusto.chinacloudapi.cn**。 该命令要求对相关数据库具有[数据库管理](kusto/management/access-control/role-based-authorization.md)权限。 
* 由于清除过程的性能影响，调用方需要修改数据架构，以便使最小的表包含相关数据，并对每个表执行批处理命令，以减轻清除过程中产生的较大 COGS 影响。
* 清除命令的 `predicate` 参数用于指定要清除的记录。 `Predicate` 大小限制为 63 KB。 

## <a name="next-steps"></a>后续步骤

* [Azure 数据资源管理器中的数据清除](kusto/concepts/data-purge.md)
