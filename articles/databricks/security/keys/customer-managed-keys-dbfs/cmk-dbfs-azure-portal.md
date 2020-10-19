---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/15/2020
title: 使用 Azure 门户为 DBFS 配置客户管理的密钥 - Azure Databricks
description: 了解如何使用 Azure 门户配置自己的加密密钥以加密 DBFS 存储帐户。
ms.openlocfilehash: 1bcfe4de170b8e9e6d043d921108f351b6e529cf
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937706"
---
# <a name="configure-customer-managed-keys-for-dbfs-using-the-azure-portal"></a>使用 Azure 门户为 DBFS 配置客户管理的密钥

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> 此功能仅在 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)中提供。

可以使用 Azure 门户来配置自己的加密密钥以加密 DBFS 根存储帐户。 必须使用 [Azure Key Vault](/key-vault/general/overview) 存储密钥。

若要详细了解用于 DBFS 的客户管理的密钥，请参阅[为 DBFS 根配置客户管理的密钥](index.md)。

## <a name="create-a-key-in-azure-key-vault"></a>在 Azure Key Vault 中创建密钥

> [!NOTE]
>
> 如果已在 Azure Databricks 工作区所在的区域和所在的 Azure Active Directory (Azure AD) 租户中拥有密钥保管库，你可以跳过此过程中的第一步。 但请注意，当你使用 Azure 门户为 DBFS 根加密分配客户管理的密钥时，默认情况下系统将为密钥保管库启用“软删除”和“不清除”属性。 有关这些属性的详细信息，请参阅 [Azure Key Vault 软删除概述](/key-vault/general/soft-delete-overview)。

1. 按照[快速入门：使用 Azure 门户在 Azure Key Vault 中设置和检索密钥](/key-vault/keys/quick-create-portal)中的说明创建密钥保管库。

   Azure Databricks 工作区与密钥保管库必须在同一区域和同一 Azure Active Directory (Azure AD) 租户中，但可以在不同的订阅中。

2. 继续按照该快速入门中的说明在此密钥保管库中创建一个密钥。

   DBFS 根存储支持 2048、3072 和 4096 大小的 RSA 和 RSA-HSM 密钥。 有关密钥的详细信息，请参阅[关于 Key Vault 密钥](/key-vault/keys/about-keys)。

3. 创建密钥后，将**密钥标识符**复制并粘贴到文本编辑器中。 为 Azure Databricks 配置密钥时将需要用到它。

## <a name="encrypt-the-dbfs-root-storage-account-using-your-key"></a>使用密钥加密 DBFS 根存储帐户

1. 在 Azure 门户中转到 Azure Databricks 服务资源。
2. 在左侧菜单中的“设置”下，选择“加密”。

   > [!div class="mx-imgBorder"]
   > ![Azure Databricks 的加密选项](../../../_static/images/security/databricks-encryption-azure.png)

3. 选择“使用自己的密钥”，输入密钥的“密钥标识符”，并选择包含密钥的“订阅”。

   > [!div class="mx-imgBorder"]
   > ![在 Azure 门户中启用客户管理的密钥](../../../_static/images/security/byok-enable-azure.png)

4. 单击“保存”以保存密钥配置。

   > [!NOTE]
   >
   > 只有具有[密钥保管库参与者角色](/key-vault/general/overview-security#managing-administrative-access-to-key-vault)或密钥保管库的更高角色的用户可以保存。

启用加密后，系统会在密钥保管库上启用“软删除”和“清除保护”，在 DBFS 根上创建一个托管标识，并在密钥保管库中为此标识添加一个访问策略。

## <a name="regenerate-rotate-keys"></a>重新生成（轮换）密钥

重新生成密钥时，必须返回到 Azure Databricks 服务资源中的“加密”页，使用新的密钥标识符更新“密钥标识符”字段，然后单击“保存”。 这适用于同一密钥和新密钥的新版本。

> [!IMPORTANT]
>
> 如果删除了用于加密的密钥，则无法访问 DBFS 根中的数据。 可以使用 Azure Key Vault API 来[恢复已删除的密钥](https://docs.microsoft.com/rest/api/keyvault/recoverdeletedkey/recoverdeletedkey)。