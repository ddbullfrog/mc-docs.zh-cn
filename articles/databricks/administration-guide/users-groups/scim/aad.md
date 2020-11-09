---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 为 Microsoft Azure Active Directory 配置 SCIM 预配 - Azure Databricks
description: 了解如何使用 Microsoft Azure Active Directory 将用户预配到 Azure Databricks。
ms.openlocfilehash: b6ebf6112fc06d77e330ffad54c6ea62e2f0483e
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106502"
---
# <a name="configure-scim-provisioning-for-microsoft-azure-active-directory"></a><a id="configure-scim-provisioning-for-microsoft-azure-active-directory"> </a><a id="provisioning-aad"> </a>为 Microsoft Azure Active Directory 配置 SCIM 预配

若要使用 Azure Active Directory (Azure AD) 启用到 Azure Databricks 的预配，必须为每个 Azure Databricks 工作区创建一个企业应用程序。

> [!NOTE]
>
> 对配置进行预配完全独立于为 Azure Databricks 工作区设置身份验证和条件访问的过程。 Azure Databricks 的身份验证由 Azure Active Directory 使用 OpenID Connect 协议流自动处理。 可以在服务级别建立条件访问，以便创建规则来要求进行多重身份验证或将登录限制到本地网络。 有关说明，请参阅[条件访问](../../access-control/conditional-access.md)。

## <a name="requirements"></a><a id="prereq"> </a><a id="requirements"> </a>要求

你的 Azure AD 帐户必须是高级版帐户，并且你必须是该帐户的全局管理员才能启用预配。

## <a name="create-an-enterprise-application-and-connect-to-the-azure-databricks-scim-api"></a>创建企业应用程序并连接到 Azure Databricks SCIM API

在以下示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。

1. 在 Azure Databricks 中生成一个[个人访问令牌](../../../dev-tools/api/latest/authentication.md#token-management)并复制它。 在后续步骤中，你将此令牌提供给 Azure AD。

   > [!IMPORTANT]
   >
   > 以 Azure Databricks 管理员身份生成此令牌，该管理员将不由 Azure AD 企业应用程序管理。 可以使用 Azure AD 取消预配由此企业应用程序管理的 Azure Databricks 管理员用户，这会导致 SCIM 预配集成被禁用。

2. 在 Azure 门户中，转到“Azure Active Directory”>“企业应用程序”。
3. 单击应用程序列表上方的“+ 新建应用程序”。 在“从库中添加”下，搜索并选择“Azure Databricks SCIM 预配连接器”。
4. 输入应用程序的 **名称** 并单击“添加”。 请使用有助于管理员查找的名称，例如 `<workspace-name>-provisioning`。
5. 在“管理”菜单下，单击“预配”。
6. 从“预配模式”下拉菜单中，选择“自动” 。
7. 输入 **租户 URL** ：

   ```
   https://<databricks-instance>/api/2.0/preview/scim
   ```

   将 <databricks-instance> 替换为你的 Azure Databricks 部署的[工作区 URL](../../../workspace/workspace-details.md#workspace-url)。 请参阅[获取工作区、群集、笔记本、模型和作业标识符](../../../workspace/workspace-details.md)。

8. 在“机密令牌”字段中，输入在第 1 步生成的 Azure Databricks 个人访问令牌。
9. 单击“测试连接”，等待确认凭据已获得启用预配的授权的消息。
10. （可选）输入通知电子邮件，接收与 SCIM 预配严重错误有关的通知。
11. 单击“保存”  。

## <a name="assign-users-and-groups-to-the-application"></a>将用户和组分配到应用程序

1. 转到“管理”>“预配”，在“设置”下将“范围”设置为“仅同步分配的用户和组”。

   此选项仅同步分配到企业应用程序的用户和组，是我们建议你使用的方法。

   > [!NOTE]
   >
   > Azure Active Directory 不支持将嵌套组自动预配到 Azure Databricks。 只能读取和预配属于显式分配的组的直接成员的用户。 解决方法是，显式分配（或限定）包含需要预配的用户的组。 有关详细信息，请参阅[此常见问题解答](/active-directory/manage-apps/user-provisioning#does-automatic-user-provisioning-to-saas-apps-work-with-nested-groups-in-azure-ad)。

2. 若要开始将用户和组从 Azure AD 同步到 Azure Databricks，请开启“预配状态”。
3. 单击“保存”  。
4. 测试你的预配设置：
   1. 转到“管理”>“用户和组”。
   1. 添加一些用户和组。 单击“添加用户”，选择用户和组，然后单击“分配”按钮。
   1. 等待几分钟，检查是否已将用户和组添加到 Azure Databricks 工作区。

当 Azure AD 计划下一次同步时，将自动预配你添加和分配的任何其他用户和组。

> [!IMPORTANT]
>
> 不要分配其机密令牌（持有者令牌）已用于设置此企业应用程序的 Azure Databricks 管理员。

## <a name="provisioning-tips"></a>预配提示

* 启用预配之前 Azure Databricks 中已存在的用户和组在预配同步时展示以下行为：
  * 如果它们也存在于此 Azure AD 企业应用程序中，则会进行合并。
  * 如果它们不存在于此 Azure AD 企业应用程序中，则会被忽略。
* 为用户删除组成员身份后，单独分配并通过组中的成员身份复制的用户权限会保留。
* 使用 Azure Databricks 管理控制台直接从 Azure Databricks 工作区中删除的用户：
  * 失去对该 Azure Databricks 工作区的访问权限，但仍可访问其他 Azure Databricks 工作区。
  * 不会使用 Azure AD 预配再次同步，即使它们保留在企业应用程序中。
* 初始 Azure AD 同步将在你启用预配后立即触发。 后续同步每 20-40 分钟触发一次，具体取决于应用程序中的用户和组的数目。 请参阅 Azure AD 文档中的[预配摘要报告](/active-directory/manage-apps/check-status-user-account-provisioning#provisioning-summary-report)。
* “admins”组是 Azure Databricks 中的保留组，无法删除。
* 无法在 Azure Databricks 中重命名组；请勿尝试在 Azure AD 中对它们重命名。
* 可以使用 Azure Databricks [组 API](../../../dev-tools/api/latest/groups.md) 或[组 UI](../groups.md) 获取任何 Azure Databricks 组的成员列表。
* 无法更新 Azure Databricks 用户名和电子邮件地址。

## <a name="troubleshooting"></a>疑难解答

**用户和组不同步**

此问题可能是因为其个人访问令牌用来连接到 Azure AD 的 Azure Databricks 管理员用户失去了管理员状态或具有的令牌无效：请以该用户身份登录到 Azure Databricks 管理控制台，并验证你是否仍是管理员以及你的访问令牌是否仍然有效。

另一种可能性是你尝试同步嵌套组，Azure AD 自动预配不支持此类组。 请参阅[此常见问题解答](/active-directory/manage-apps/user-provisioning#does-automatic-user-provisioning-to-saas-apps-work-with-nested-groups-in-azure-ad)。

**初始同步后，用户和组不进行同步**

初始同步后，Azure AD 不会立即根据用户和组分配的更改进行同步。 它在延迟一段时间后才安排与应用程序的同步（具体取决于用户和组的数目）。 你可以转到企业应用程序的“管理”>“预配”，选择“清除当前状态并重启同步”以启动立即同步。