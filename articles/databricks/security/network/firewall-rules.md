---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/30/2020
title: 配置域名防火墙规则 - Azure Databricks
description: 了解如何为 Azure Databricks 工作区配置域名防火墙规则。
ms.openlocfilehash: 99f1e4b41d026b706aef91422af69e9505ba189b
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937702"
---
# <a name="configure-domain-name-firewall-rules"></a>配置域名防火墙规则

如果企业防火墙根据域名来阻止流量，你必须允许 HTTPS 和 WebSocket 流量访问 Azure Databricks 域名，以确保访问 Azure Databricks 资源。 你可以在两个选项之间进行选择，一个选项在权限方面更宽松但更易于配置，另一个选项特定于你的工作区域。

## <a name="option-1-allow-traffic-to-databricksazurecn"></a>选项 1：允许流量流向 `*.databricks.azure.cn`

更新防火墙规则，使之允许 HTTPS 和 WebSocket 流量流向 `*.databricks.azure.cn`。 此选项在权限方面比选项 2 更宽松，但它省去了为帐户中的每个 Azure Databricks 工作区更新防火墙规则的工作量。

## <a name="option-2-allow-traffic-to-your-azure-databricks-workspaces-only"></a>选项 2：仅允许流量流向 Azure Databricks 工作区

如果选择为帐户中的每个工作区配置防火墙规则，则必须：

1. 确定工作区的域。

   每个 Azure Databricks 资源有两个唯一的域名。 可以通过转到 Azure 门户中的 Azure Databricks 资源来找到第一个域名。

   > [!div class="mx-imgBorder"]
   > ![工作区 URL](../../_static/images/workspace/azure-workspace-url.png)

   URL 字段以 `https://adb-<digits>.<digits>.databricks.azure.cn` 格式显示 URL，例如 `https://adb-1666506161514800.0.databricks.azure.cn`。 删除 `https://` 以获取第一个域名。

   第二个域名与第一个域名完全相同，只不过它具有 `adb-dp-` 前缀而不是 `adb-`。 例如，如果第一个域名为 `adb-1666506161514800.0.databricks.azure.cn`，则第二个域名为 `adb-dp-1666506161514800.0.databricks.azure.cn`。

2. 更新防火墙规则。

   更新防火墙规则以允许 HTTPS 和 WebSocket 流量流向步骤 1 中确定的两个域。