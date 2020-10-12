---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 为 DBFS 根配置客户管理的密钥 - Azure Databricks
description: 了解如何使用自己的加密密钥来加密 DBFS 存储帐户。
ms.openlocfilehash: 16c0d2fedfdb63cbe0ca58524f44b47f609268f8
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937717"
---
# <a name="configure-customer-managed-keys-for-dbfs-root"></a>为 DBFS 根配置客户管理的密钥

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 此功能仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中提供。

[Databricks 文件系统](../../../data/databricks-file-system.md) (DBFS) 是一个装载到 Azure Databricks 工作区的分布式文件系统，可以在 Azure Databricks 群集上使用。 DBFS 在 Azure Databricks 工作区的受管理资源组中实现为存储帐户。 DBFS 中的默认存储位置称为 [DBFS 根](../../../data/databricks-file-system.md#dbfs-root)。 默认情况下，存储帐户使用 Microsoft 管理的密钥进行加密。

通过用于 DBFS 根的客户管理的密钥 (CMK)，你可以使用自己的加密密钥来加密 DBFS 存储帐户。 指定客户托管密钥时，该密钥用于保护和控制对数据加密密钥的访问。 使用客户托管密钥可以更灵活地管理访问控制。

必须使用 [Azure Key Vault](/key-vault/general/overview) 来存储客户管理的密钥。 可以创建自己的密钥并将其存储在密钥保管库中，也可以使用 Azure 密钥保管库 API 来生成密钥。

可通过三种方式为 DBFS 存储启用客户管理的密钥：

* [使用 Azure 门户为 DBFS 配置客户管理的密钥](cmk-dbfs-azure-portal.md)
* [使用 Azure CLI 为 DBFS 配置客户管理的密钥](cmk-dbfs-azure-cli.md)
* [使用 PowerShell 为 DBFS 配置客户管理的密钥](cmk-dbfs-powershell.md)