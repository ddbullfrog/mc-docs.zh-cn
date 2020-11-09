---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/17/2020
title: 管理个人访问令牌 - Azure Databricks
description: 为 Azure Databricks REST API 客户端管理基于令牌的身份验证。
ms.openlocfilehash: 5f8d2a335f4e07620fc614b04d646fa6d68f7e6d
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106489"
---
# <a name="manage-personal-access-tokens"></a>管理个人访问令牌

若要向 Azure Databricks REST API 进行身份验证，用户可以创建个人访问令牌并在其 REST API 请求中使用它。 令牌有一个可选的到期日期，可以被撤销。 请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../../dev-tools/api/latest/authentication.md)。

默认情况下，将为在 2018 年或之后创建的所有 Azure Databricks 工作区启用使用个人访问令牌的功能。 工作区管理员可以为所有工作区启用或禁用个人令牌访问权限，不管创建日期是什么时候。

工作区管理员还可以监视令牌、控制哪些非管理员用户可以创建令牌，以及为新令牌设置最大生存期。

> [!NOTE]
>
> 你还可以让 Azure Databricks 用户使用 [Azure Active Directory](/dev-tools/api/latest/aad/) 令牌而不是 Azure Databricks 个人访问令牌进行 REST API 访问。 如果你的工作区使用 Azure Active Directory 令牌，则本文中的说明不适用。

为你的工作区[启用](#enable-tokens)了生成个人访问令牌的功能时，默认情况下，Azure Databricks 工作区中的所有用户都可以生成个人访问令牌来访问 Azure Databricks REST API，并且可以使用所需的任何到期日期（包括无限生存期）来生成这些令牌。

作为 Azure Databricks 管理员，你可以使用[令牌管理 API](../../dev-tools/api/latest/token-management.md) 和[权限 API](../../dev-tools/api/latest/permissions.md) 来更精细地控制令牌使用。 API 在每个工作区实例上发布。 若要了解如何访问 API 以及如何向其进行身份验证，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../../dev-tools/api/latest/authentication.md)。 必须以 Azure Databricks 管理员身份访问 API。

对于某些任务，还可以使用管理控制台。

下表指示了可以使用 Web 应用程序执行的任务，以及可以使用 REST API 执行的任务。 对于显示为“是”的单元格，单击此字可以查看相关文档。

| 任务                         | 说明                                                                                                                                                                                                                                                                                                                                           | 管理控制台             | REST API                  |
|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------|---------------------------|
| 启用/禁用               | 启用或禁用此工作区的所有令牌                                                                                                                                                                                                                                                                                                       | **[是](#enable-tokens)** | **[是](#enable-tokens)** |
| 控制谁可以使用令牌   | 将个人访问令牌的创建和使用限制为此工作区中的指定用户和组。 如果你撤销用户创建和使用令牌的权限，则该用户的现有令牌也会被撤销。                                                                                                                                             | **[是](#manage)**        | **[是](#manage)**        |
| 新令牌的最长生存期。 | 设置此工作区中新令牌的最长生存期                                                                                                                                                                                                                                                                                              | 否                        | **[是](#lifetime)**      |
| 管理现有令牌       | 对于现有令牌，获取令牌创建者、到期日期和用户提供的令牌说明。 撤销不应再有权访问 Azure Databricks API 的用户的令牌。 通过监视和控制令牌创建情况，可以降低丢失令牌的风险，或降低可能导致从工作区渗透数据的长效令牌的风险。 | 否                        | **[是](#manage)**        |

请参阅下图，了解通过 REST API 管理和使用令牌的典型令牌管理流。 请参阅本主题中的表，了解你还可以使用管理控制台的一组任务。

> [!div class="mx-imgBorder"]
> ![令牌管理](../../_static/images/access-control/token-management.png)

## <a name="enable-or-disable-token-based-authentication-for-the-workspace"></a><a id="enable-or-disable-token-based-authentication-for-the-workspace"> </a><a id="enable-tokens"> </a>为工作区启用或禁用基于令牌的身份验证

默认情况下，将为在 2018 年或之后创建的所有 Azure Databricks 工作区启用基于令牌的身份验证。 你可以在管理控制台中更改此设置。 若要指定允许哪些用户使用令牌，请参阅[控制谁可以使用或创建令牌](#permissions)。

若要为工作区启用或禁用个人访问令牌，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。
3. 若要启用访问权限，请单击 **个人访问令牌** 旁边的“启用”按钮。 若要禁用访问权限，请单击“禁用”按钮。
4. 单击“确认”以确认更改。 此更改可能需要几秒钟的时间才能生效。

若要为 REST API 请求使用基于令牌的身份验证，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../../dev-tools/api/latest/authentication.md)。

为工作区禁用基于令牌的身份验证时，不会删除任何令牌。 如果以后重新启用令牌，则任何未过期的令牌会立即变得可供使用。

如果要对部分用户禁用令牌访问权限，请始终为该工作区启用基于令牌的身份验证，并为用户和组设置细化的权限。 请参阅[控制谁可以使用或创建令牌](#permissions)。

你还可以使用 REST API 进行此更改。 若要为工作区启用或禁用令牌管理功能，请调用[令牌 API 的工作区配置](../../_static/api-refs/token-management-azure.yaml) (`PATCH /workspace-conf`)。 在 JSON 请求正文中，将 `enableTokensConfig` 指定为 `true`（已启用）或 `false`（已禁用）。

例如，启用该功能：

```bash
curl -X PATCH -n \
  https://<databricks-instance>/api/2.0/workspace-conf \
  -d '{
    "enableTokensConfig": "true",
    }'
```

## <a name="control-who-can-use-or-create-tokens"></a><a id="control-who-can-use-or-create-tokens"> </a><a id="permissions"> </a>控制谁可以使用或创建令牌

用户可以有下列令牌权限之一：

* **无权限**
* **可以使用** – 对于在 Azure Databricks 平台版本 3.28 发布之后（2020 年 9 月 9 日至 15 日）创建的工作区，默认设置是没有用户有“可以使用”权限。 管理员必须显式授予这些权限，无论是向整个 `users` 组授予还是按用户或按组授予。

  > [!IMPORTANT]
  >
  > 在 3.28 发布之前创建的工作区会保留已有的权限。 默认情况下，所有用户都有“可以使用”权限。 管理员可以撤销该组权限，并将其授予其他组或单独的非管理员用户。 请参阅[删除权限](#selective-access)。

* **可以管理** – `admins` 组中的用户默认具有此权限，你无法撤销它。 不能向其他组授予此权限。 API 强制实施这些规则。

此表列出了每个与令牌相关的任务所需的权限：

| 任务                           | 无权限           | 可以使用                   | 可管理                |
|--------------------------------|---------------------------|---------------------------|---------------------------|
| 创建令牌                 | –                         | 是                       | 是                       |
| 使用令牌进行身份验证 | –                         | 是                       | 是                       |
| 撤销你自己的令牌          | –                         | 是                       | 是                       |
| 撤销任何用户的令牌 **     | –                         | –                         | 是                       |
| 列出所有令牌 **             | –                         | –                         | 是                       |
| 修改令牌权限 **_   | –                         | –                         | 是                       |

标有 _* 的操作需要[令牌管理 API](../../dev-tools/api/latest/token-management.md)。

标有 **_ 的操作可以在管理控制台中执行，也可以通过[权限 API](../../dev-tools/api/latest/permissions.md) 来执行。 请参阅[控制谁可以使用或创建令牌](#permissions)。

### <a name="manage-token-permissions-using-the-admin-console"></a><a id="get-all-permissions"> </a><a id="manage-token-permissions-using-the-admin-console"> </a>使用管理控制台管理令牌权限

若要使用管理控制台管理工作区的令牌权限，请执行以下操作：

1. 转到[管理控制台](../admin-console.md)。
2. 选择“访问控制”选项卡。
3. 如果禁用了基于令牌的身份验证，请单击 **个人访问令牌** 旁边的“启用”按钮。 单击“确认”以确认更改。 此更改可能需要几秒钟的时间才能生效。

   > [!div class="mx-imgBorder"]
   > ![令牌启用](../../_static/images/tokens/tokens-enable.png)

4. 单击“权限设置”按钮以打开令牌权限编辑器。

   > [!div class="mx-imgBorder"]
   > ![令牌权限编辑器](../../_static/images/access-control/tokens-permissions-editor.png)

5. 添加或删除权限

   > [!IMPORTANT]
   >
   > 对于在 Azure Databricks 平台版本 3.28 发布之后（2020 年 9 月 9 日至 15 日）创建的工作区，默认设置是没有用户有“可以使用”权限。 管理员必须显式授予这些权限，无论是向整个 `users` 组授予还是按用户或按组授予。 在 3.28 发布之前创建的工作区保留已有的权限。 默认情况下，所有用户都有“可以使用”权限。 管理员可以撤销该组权限分配，并将其授予其他组或单独的非管理员用户。

   如果 `users` 组有“可以使用”权限，并且你想要对非管理员用户应用更精细的访问权限，则可以通过单击“用户”行中“权限”下拉列表旁边的 **X** ，从“用户”组中删除“可以使用”权限。  

   若要向其他实体授予权限，请选择你要向其授予访问权限的每个用户或组。 从“选择用户或组…”下拉列表中选择一个用户或组， 选择“可以使用”，然后单击“+ 添加”按钮。 在以下示例中，管理员已删除“用户”组的访问权限，并向“数据科学 B2”组授予访问权限。

   > [!div class="mx-imgBorder"]
   > ![令牌权限编辑器](../../_static/images/access-control/tokens-permissions-editor-notallusers.png)

   管理员组具有“可以管理”权限，你无法更改这些权限，也无法将“可以管理”分配给管理员组以外的任何实体。

6.  以保存更改。

   > [!WARNING]
   >
   > 保存更改后，以前具有“可以使用”或“可以管理”权限但现在不再具有任一权限的任何用户会被拒绝访问基于令牌的身份验证，并且其活动令牌会被立即删除（撤销）。 无法检索已删除的令牌。

### <a name="manage-token-permissions-using-the-permissions-api"></a>使用权限 API 管理令牌权限

#### <a name="get-all-token-permissions-for-the-workspace"></a>获取工作区的所有令牌权限

若要为工作区的所有 Azure Databricks 用户、Azure Databricks 组和 Azure 服务主体获取令牌权限，请调用[为工作区 API 获取所有令牌权限](../../_static/api-refs/token-management-azure.yaml) (`GET /permissions/authorization/tokens`)。

响应包括一个 `access_control_list` 数组。 每个元素都是一个用户对象、组对象或服务主体对象。 每个用户都有一个适用于该类型的标识字段：用户具有一个 `user_name` 字段，组具有一个 `group_name` 字段，服务主体具有一个 `service_principal_name` 字段。 所有元素都有一个 `all_permissions` 字段，该字段指定授予的权限级别（`CAN_USE` 或 `CAN_MANAGE`）。

例如：

```bash
curl -n -X GET "https://<databricks-instance>/api/2.0/preview/permissions/authorization/tokens"
```

示例响应:

```json
{
  "object_id": "authorization/tokens",
  "object_type": "tokens",
  "access_control_list": [
    {
      "user_name": "jsmith@example.com",
      "all_permissions": [
        {
          "permission_level": "CAN_USE",
          "inherited": false
        }
      ]
    }
  ]
}
```

#### <a name="set-token-permissions"></a>设置令牌权限

若要设置令牌权限，请调用[设置令牌权限 API](../../_static/api-refs/token-management-azure.yaml) (`PATCH /permissions/authorization/tokens`)。

你可以在一个或多个用户、组或 Azure 服务主体上设置权限。 对于每个用户，你需要知道电子邮件地址，这是在 `user_name` 请求属性中指定的。 对于每个组，请在 `group_name` 属性中指定组名称。 对于 Azure 服务主体，请在 `service_principal_name` 属性中指定服务主体名称。

未显式提及的实体（例如用户或组）不会直接受此请求影响，但是对组成员身份的更改可能会间接影响用户访问权限。

仅可通过此 API 授予权限，无法通过它撤销权限。

例如，以下示例向用户 [jsmith@example.com](mailto:jsmith@example.com) 和组 mygroup 授予访问权限。

```bash
curl -n -X PATCH "https://<databricks-instance>/api/2.0/preview/permissions/authorization/tokens"
  -d '{
    "access_control_list": [
      {
        "user_name": "jsmith@example.com",
        "permission_level": "CAN_USE",
      },
      {
        "group_name": "mygroup",
        "permission_level": "CAN_USE",
      }
    ]
  }'
```

示例响应:

```json
{
  "access_control_list": [
    {
      "user_name": "jsmith@example.com",
      "all_permissions": [
        {
          "permission_level": "CAN_USE",
          "inherited": false
        }
      ]
    },
    {
      "group_name": "mygroup",
      "all_permissions": [
        {
          "permission_level": "CAN_USE",
          "inherited": false
        }
      ]
    }
  ]
}
```

如果要在一个请求中为工作区中的所有实体设置令牌权限，请使用[更新所有权限 API](../../_static/api-refs/token-management-azure.yaml) (`PUT /permissions/authorization/tokens`)。 请参阅[删除权限](#selective-access)

#### <a name="remove-permissions"></a><a id="remove-permissions"> </a><a id="selective-access"> </a>删除权限

> [!NOTE]
>
> 对于在 Azure Databricks 平台版本 3.28 发布之后（2020 年 9 月 9 日至 15 日）创建的工作区，默认设置是没有用户有“可以使用”权限。 管理员必须显式授予这些权限，无论是向整个 `users` 组授予还是按用户或按组授予。 在 3.28 发布之前创建的工作区保留已有的权限。 默认情况下，所有用户都有“可以使用”权限。 管理员可以撤销该组权限分配，并将其授予其他组或单独的非管理员用户。

若要删除所有或部分非管理员用户的权限，请使用[更新所有权限 API](../../_static/api-refs/token-management-azure.yaml) (`PUT /permissions/authorization/tokens`)，这要求你为被授予了整个工作区的权限的所有对象指定完整的权限集。

如果要授予部分非管理员用户创建和使用令牌的权限，请执行下述所有三项操作：

* 向用户、组和 Azure 服务主体 **授予** `CAN_USE` 权限。
* 如果只想向某些非管理员用户授权，请 **不要** 向内置的 `users` 组授予 `CAN_USE` 权限。 你可以选择将权限分配给此组，在这种情况下，所有非管理员用户都可以创建和使用令牌。
* 向内置的 `admins` 组 **授予** `CAN_MANAGE` 权限。 这是 API 的要求。

> [!WARNING]
>
> 请求成功后，如果用户或 Azure 服务主体没有直接或间接通过组获得令牌权限，系统会立即删除其令牌。 无法检索已删除的令牌。

以下示例根据 API 的要求向 `field-automation-group` 组授予“可以使用”令牌权限，忽略 `users`（所有用户）组的权限，并将 `CAN_MANAGE` 权限授予 `admins` 组。 不在 `field-support-engineers` 组中的任何非管理员用户会失去创建令牌所需的访问权限，系统会立即删除（撤销）其现有令牌。

```bash
curl -n -X PUT "https://<databricks-instance>/api/2.0/preview/permissions/authorization/tokens"
  -d '{
    "access_control_list": [
      {
        "group_name": "field-automation-group",
        "permission_level": "CAN_USE",
      },
      {
        "group_name": "admins",
        "permission_level": "CAN_MANAGE",
      },
    ]
  }'
```

## <a name="set-maximum-lifetime-of-new-tokens-rest-api-only"></a><a id="lifetime"> </a><a id="set-maximum-lifetime-of-new-tokens-rest-api-only"> </a>设置新令牌的最长生存期（仅限 REST API）

使用[令牌生存期管理 API](../../_static/api-refs/token-management-azure.yaml) 管理此工作区中的新令牌的最长生存期。

若要设置新令牌的最长生存期，请调用[设置新令牌 API 的最长令牌生存期](../../_static/api-refs/token-management-azure.yaml) (`PATCH /workspace-conf`)。 将 `maxTokenLifetimeDays` 设置为新令牌的最长令牌生存期（以天为单位的整数）。 如果将其设置为零，则允许新令牌无生存期限制。

例如：

```bash
curl -n -X PATCH "https://<databricks-instance>/api/2.0/workspace-conf"
  -d '{
  "maxTokenLifetimeDays": "90"
  }'
```

> [!WARNING]
>
> 此限制仅适用于新令牌。 若要查看现有令牌，请参阅[获取令牌 API](../../_static/api-refs/token-management-azure.yaml)。

若要获取工作区的新令牌的最长生存期，请调用[获取新令牌 API 的最长令牌生存期](../../_static/api-refs/token-management-azure.yaml) (`GET /workspace-conf`)，并将 `keys=maxTokenLifetimeDays` 作为查询参数传递。 响应包含一个 `maxTokenLifetimeDays` 属性，该属性是新令牌的最长令牌生存期（以天为单位的整数）。 如果它是零，则允许新令牌无生存期限制。

例如：

```bash
curl -n -X GET "https://<databricks-instance>/api/2.0/workspace-conf?keys=maxTokenLifetimeDays"
```

示例响应:

```json
{
    "maxTokenLifetimeDays": "90"
}
```

## <a name="monitor-and-revoke-tokens-rest-api-only"></a><a id="manage"> </a><a id="monitor-and-revoke-tokens-rest-api-only"> </a>监视和撤销令牌（仅限 REST API）

使用[令牌管理 API](../../_static/api-refs/token-management-azure.yaml) 管理工作区中的现有令牌。

### <a name="get-tokens-for-the-workspace"></a><a id="get-tokens"> </a><a id="get-tokens-for-the-workspace"> </a>获取工作区的令牌

若要获取工作区的令牌，请调用[获取所有令牌 API](../../_static/api-refs/token-management-azure.yaml) (`GET /token-management/tokens`)。 响应包括一个 `token_infos` 数组。 每个元素代表一个令牌，包含的字段有：ID (`token_id`)、创建时间 (`creation_time`)、到期时间 (`expiry_time`)、说明 (`comment`) 和创建它的用户（ID 为 `created_by_id`，用户名为 `created_by_username`）。 你可以使用 [SCIM 获取用户 API](../../dev-tools/api/latest/scim/scim-users.md) (`GET /scim/v2/Users/{id}`) 来了解有关用户的详细信息。

若要按用户筛选结果，请设置请求正文属性 `created_by_id`（适用于 ID）或 `created_by_username`（适用于用户名）。 可以使用 [SCIM 获取用户 API](../../dev-tools/api/latest/scim/scim-users.md) (`GET /scim/v2/Users`) 从显示名称获取用户 ID

例如：

```bash
curl -n -X GET "https://<databricks-instance>/api/2.0/token-management/tokens"
  -d '{
  "created_by_id": "1234567890"
  }'
```

示例响应:

```json
{
  "token_infos": [
    {
      "token_id": "<token-id>",
      "creation_time": 1580265020299,
      "expiry_time": 1580265020299,
      "comment": "This is for ABC division's automation scripts.",
      "created_by_id": 1234567890,
      "created_by_username": "jsmith@example.com"
    }
  ]
}
```

或者，使用[获取令牌 API](../../_static/api-refs/token-management-azure.yaml) (`GET /token-management/tokens/{token_id}`) 获取特定令牌

### <a name="delete-revoke-a-token"></a>删除（撤销）令牌

1. 找到令牌 ID。 请参阅[获取工作区的令牌](#get-tokens)。
2. 调用[删除令牌 API](../../_static/api-refs/token-management-azure.yaml) (`DELETE /token-management/tokens`)。 在路径中传递令牌 ID。

例如：

```bash
curl -n -X DELETE "https://<databricks-instance>/api/2.0/token-management/tokens/<token-id>"
```