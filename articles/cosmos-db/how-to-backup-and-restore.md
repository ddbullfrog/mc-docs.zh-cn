---
title: 如何从备份还原 Azure Cosmos DB 数据
description: 本文介绍如何从备份还原 Azure Cosmos DB 数据，如何联系 Azure 支持人员还原数据，以及数据恢复后要执行的步骤。
ms.service: cosmos-db
ms.topic: how-to
origin.date: 08/24/2020
author: rockboyfor
ms.date: 09/28/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.reviewer: sngun
ms.openlocfilehash: 8d294a35d30a48e752e8b7d3d4d6b1ad20b266ac
ms.sourcegitcommit: b9dfda0e754bc5c591e10fc560fe457fba202778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91246772"
---
# <a name="restore-data-from-a-backup-in-azure-cosmos-db"></a>如何在 Azure Cosmos DB 中从备份还原数据

如果意外删除了数据库或容器，可以[提交支持工单](https://support.azure.cn/support/support-azure/)或[联系 Azure 支持](https://support.azure.cn/support/contact/)，以便从自动联机备份中还原数据。 Azure 支持仅适用于选定的计划，例如“标准”、“开发人员”以及更高级别的计划 。 不适用于基本计划  。 若要了解不同的支持方案，请参阅 [Azure 支持计划](https://www.azure.cn/support/plans/)页。

若要还原备份的特定快照，Azure Cosmos DB 要求在该快照的备份周期的持续时间内可用。

## <a name="request-a-restore"></a>请求还原

在请求还原之前，应该了解以下详细信息：

* 准备好订阅 ID。

* 应根据数据被意外删除或修改的方式，准备好提供其他信息。 建议提前准备可用的信息，从而尽量减少可能在某些有时效的情况下造成不良影响的来回传输。

* 如果删除了整个 Azure Cosmos DB 帐户，则需要提供删除的帐户的名称。 如果创建了同名的另一个帐户，请与支持团队共享该帐户，因为这有助于确定要选择的正确帐户。 建议为删除的每个帐户提交不同的支持工单，因为这可以最大限度地减少还原状态的混乱。

* 如果删除了一个或多个数据库，应提供 Azure Cosmos 帐户及 Azure Cosmos 数据库名，并指定是否存在同名的新数据库。

* 如果删除了一个或多个容器，应提供 Azure Cosmos 帐户名、数据库名和容器名。 并指定是否存在同名容器。

* 如果意外删除或损坏了数据，则应在 8 小时内联系 [Azure 支持](https://support.azure.cn/support/contact/)，以便 Azure Cosmos DB 团队帮助你从备份中还原数据。 在你创建支持请求以申请还原数据之前，请确保[将帐户的备份保留期延长](online-backup-and-restore.md)到至少 7 天。最好在发生此事件的 8 小时内延长保留期。 这样，Azure Cosmos DB 支持团队才有足够的时间来还原你的帐户。

除了 Azure Cosmos 帐户名、数据库名、容器名以外，还应指定数据可以还原到的时间点。 务必尽量精确，因为这有助于我们确定当时可用的最佳备份。 **指定 UTC 时间也很重要。**

<!--MOONCAKE CUSTOMIZE on SUPPORT REQUEST-->

下面的屏幕截图说明如何为容器（集合/图/表）创建通过 Azure 门户还原数据的支持请求。 提供其他详细信息（例如数据类型、还原目的、删除数据的时间），以帮助我们设置请求的优先级。

:::image type="content" source="./media/how-to-backup-and-restore/backup-support-request-portal.png" alt-text="使用 Azure 门户创建备份支持请求":::

<!--MOONCAKE CUSTOMIZE on SUPPORT REQUEST-->

## <a name="post-restore-actions"></a>还原后的操作

还原数据后，你会收到有关新帐户名（通常采用 `<original-name>-restored1` 格式）和帐户要还原到的时间的通知。 还原的帐户与原始帐户具有相同的预配吞吐量、索引策略，并且二者位于同一区域。 角色为订阅管理员或共同管理员的用户可以看到还原的帐户。

还原数据后，应检查并验证还原的帐户中的数据，确保其中包含你需要的版本。 如果一切正常，应该使用 [Azure Cosmos DB 更改源](change-feed.md)或 [Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)将数据迁移回原始帐户。

建议在迁移数据之后立即删除容器或数据库。 如果不删除已还原的数据库或容器，它们将在请求单位、存储和流出量方面产生成本。

## <a name="next-steps"></a>后续步骤

接下来，可以通过以下文章学习如何将数据迁移回原始帐户：

* 若要提出还原请求，请联系 Azure 支持，并[从 Azure 门户提交票证](https://support.azure.cn/support/support-azure/)
* [使用 Cosmos DB 更改源](change-feed.md)将数据移动到 Azure Cosmos DB。

* [使用 Azure 数据工厂](../data-factory/connector-azure-cosmos-db.md)将数据移动到 Azure Cosmos DB。

<!-- Update_Description: update meta properties, wording update, update link -->