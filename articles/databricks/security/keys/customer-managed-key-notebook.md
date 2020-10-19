---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/26/2020
title: 为笔记本启用客户管理的密钥 - Azure Databricks
description: 了解如何为 Azure Databricks 笔记本配置你自己的密钥（客户管理的密钥）。
ms.openlocfilehash: ddaaada5d395a92d3d12ee4e7648e7bc2c4f55c9
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937704"
---
# <a name="enable-customer-managed-keys-for-notebooks"></a>为笔记本启用客户管理的密钥

Azure Databricks 工作区包含一个托管在 Azure Databricks 管理的虚拟网络中的管理平面，还包含一个部署在客户管理的虚拟网络中的数据平面。 控制平面将所有用户的笔记本和关联的笔记本结果存储在数据库中。 默认情况下，系统使用不同的加密密钥对所有笔记本和结果进行静态加密。

如果安全性和合规性要求意味着你必须拥有和管理用于加密笔记本和结果的密钥，那么你可以按照本文中的说明提供你自己的密钥，用于加密 Azure Databricks 托管控制平面中的所有数据。

> [!NOTE]
>
> 此功能并非适用于所有 Azure Databricks 订阅。 请联系 Microsoft 或 Databricks 客户代表，以申请访问权限。

## <a name="overview"></a>概述

下面是说明如何自带密钥以及该过程工作原理的概要视图：

1. 创建与 Azure Databricks 配合使用的 Azure Key Vault，并创建可与一个或多个 Azure Databricks 工作区配合使用的加密密钥。
2. 通过支持票证将密钥的资源标识符传递给 Databricks。
3. Databricks 人员在工作区的 Databricks 配置中配置密钥 ID。
4. 然后，Databricks 人员创建使用你提供的密钥包装的加密密钥。 这些加密密钥以加密方式存储在数据库中。

每次笔记本在数据库中进行读取或写入操作时，都会从数据库中读取相应的 Azure Databricks 密钥，并通过向 Key Vault 发送一个请求将该密钥解包。

由于每次读写都要往返到密钥保管库，这会显著影响性能，因此解包的加密密钥会缓存在内存中，以便执行多次读/写操作，并按定期的时间间隔从内存中驱逐，使新的读/写操作需要向密钥保管库发出请求。 如果删除或撤消密钥保管库中的密钥，则间隔时间过后，对笔记本的所有读/写操作和结果都会失败。

## <a name="limitations"></a>限制

对于当前版本，以下限制适用：

* Azure Databricks 不支持在指定密钥后更改密钥，因为没有用于配置密钥的用户界面或 API。

## <a name="enable-your-key"></a>启用密钥

1. 创建或选择要用于笔记本加密的 Key Vault 密钥。 KeyType 必须为“RSA”，但是 RSA 密钥大小和 HSM 不重要。 KeyVault 必须与 Azure Databricks 工作区位于同一租户中。
2. 在 Key Vault 设置中，单击“访问策略”。
3. 单击“新增” 。
4. 单击“选择主体”并添加“AzureDatabricks(GUID:2ff814a6-3304-4ab8-85cb-cd0e6f879c1d)”服务主体。 如果看不到该服务主体，这意味着 Key Vault 与 Azure Databricks 工作区不在同一租户中。
5. 单击“密钥权限”，然后在加密操作下添加“包装密钥”、“解包密钥”。
6. 单击“确定”。
7. 获取“密钥标识符”和“tenantId”，并使用支持票证将其发送到 Azure Databricks。
   * **密钥标识符：** “密钥”-> 单击密钥 -> 单击版本 ->“密钥标识符”。 例如，[https://kevin-testing-keyvault.vault.azure.net/keys/byok-testing/13c9c00eegb4klyuyl1ed766ad8db0b0b](https://kevin-testing-keyvault.vault.azure.net/keys/byok-testing/13c9c00eegb4klyuyl1ed766ad8db0b0b)
   * **TenantId：** “Azure Active Directory”->“属性”->“目录 ID”。 例如，`9f36a393-f0re-4280-9796-f1554a10eftg`。
8. 确认该密钥已启用，并且已将“包装密钥”和“解包密钥”作为“允许的操作”。

   > [!div class="mx-imgBorder"]
   > ![客户管理的密钥设置确认](../../_static/images/security/cmk-notebook-confirm.png)

## <a name="troubleshooting-and-best-practices"></a>故障排除和最佳做法

**意外删除**

如果在 Azure Key Vault 中删除密钥，则工作区登录就会开始失败，Azure Databricks 将无法读取任何笔记本。 为避免出现这种情况，建议启用软删除。 此选项可确保一个密钥被删除后，可以在 30 天内恢复。 如果启用了软删除，只需重新启用密钥即可解决该问题。

**丢失密钥**

如果丢失了密钥并且无法恢复它，则使用该密钥加密的所有笔记本数据都将无法恢复。