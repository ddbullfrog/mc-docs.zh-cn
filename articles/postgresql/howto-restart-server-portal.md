---
title: 重启服务器 - Azure 门户 - Azure Database for PostgreSQL - 单个服务器
description: 本文介绍了如何使用 Azure 门户重启 Azure Database for PostgreSQL - 单一服务器。
author: WenJason
ms.author: v-jay
ms.service: postgresql
ms.topic: how-to
origin.date: 5/6/2019
ms.date: 10/19/2020
ms.openlocfilehash: e501f6c7915e6b410aaa8105e7d27c4f66a5f7ac
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041750"
---
# <a name="restart-azure-database-for-postgresql---single-server-using-the-azure-portal"></a>使用 Azure 门户重启 Azure Database for PostgreSQL - 单一服务器
本主题介绍如何重启 Azure Database for PostgreSQL 服务器。 出于维护原因，可能需要重启服务器，这会在服务器执行操作时导致短暂中断。

如果服务处于繁忙状态，则会阻止重启服务器。 例如，服务可能正在处理先前请求的操作（例如缩放 vCore）。
 
完成重启所需的时间取决于 PostgreSQL 恢复过程。 若要减少重启时间，建议在重启之前尽量减少服务器上发生的活动量。

## <a name="prerequisites"></a>先决条件
若要完成本操作指南，需要：
- [Azure Database for PostgreSQL 服务器](quickstart-create-server-database-portal.md)

## <a name="perform-server-restart"></a>执行服务器重启

可通过以下步骤重启 PostgreSQL 服务器：

1. 在 [Azure 门户](https://portal.azure.cn/)中，选择 Azure Database for PostgreSQL 服务器。

2. 在服务器“概述”页的工具栏中，单击“重启”   。

   :::image type="content" source="./media/howto-restart-server-portal/2-server.png" alt-text="Azure Database for PostgreSQL - 概述 - “重启”按钮":::

3. 单击“是”以确认重启服务器  。

   :::image type="content" source="./media/howto-restart-server-portal/3-restart-confirm.png" alt-text="Azure Database for PostgreSQL - 概述 - “重启”按钮":::

4. 观察到服务器状态更改为“正在重启”。

   :::image type="content" source="./media/howto-restart-server-portal/4-restarting-status.png" alt-text="Azure Database for PostgreSQL - 概述 - “重启”按钮":::

5. 确认服务器重启成功。

   :::image type="content" source="./media/howto-restart-server-portal/5-restart-success.png" alt-text="Azure Database for PostgreSQL - 概述 - “重启”按钮":::

## <a name="next-steps"></a>后续步骤

了解[如何在 Azure Database for PostgreSQL 中设置参数](howto-configure-server-parameters-using-portal.md)