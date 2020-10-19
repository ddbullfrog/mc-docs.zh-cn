---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/17/2020
title: IP 访问列表 - Azure Databricks
description: 了解如何将 Azure Databricks 工作区访问限制为仅限授权的 IP 地址。
ms.openlocfilehash: f1925d26244f41a140bfdc67a807b9b3a40183ab
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937788"
---
# <a name="ip-access-lists"></a>IP 访问列表

> [!NOTE]
>
> 该功能需要 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)。

使用云 SaaS 应用程序的安全意识较强的企业需要限制其员工的访问权限。 身份验证有助于证明用户身份，但对用户的网络位置并没有强制要求。 从不安全的网络访问云服务会给企业带来安全风险，尤其是在用户可能已获得授权访问敏感数据或个人数据的情况下。 企业网络外围应用安全策略，并限制对外部服务（例如防火墙、代理、DLP 和日志记录）的访问，因此超出这些控制的访问被视为是不可信的。

例如，假设一名医院员工访问 Azure Databricks 工作区。 如果该员工从办公室前往了一家咖啡店，则即使该员工具有访问 Web 应用程序和 REST API 的正确凭据，医院仍可阻止与 Azure Databricks 工作区的连接。

可以配置 Azure Databricks 工作区，以便员工只通过具有安全外围的现有企业网络连接到服务。 Azure Databricks 客户可以使用 IP 访问列表功能来定义一组已批准的 IP 地址。 对 Web 应用程序和 REST API 的所有传入访问都要求用户从已授权的 IP 地址进行连接。

对于远程办公或出差的员工而言，他们可以使用 VPN 连接到公司网络，从而实现对工作区的访问。 对于上一个示例，医院可以允许员工在咖啡店中使用 VPN 来访问 Azure Databricks 工作区。

> [!div class="mx-imgBorder"]
> ![IP 访问列表概述关系图](../../_static/images/security/network/ip-access-lists-azure.png)

## <a name="flexible-configuration"></a>灵活配置

IP 访问列表功能非常灵活：

* 你自己的工作区管理员控制公共 Internet 上允许访问的 IP 地址集。 这就是所谓的允许列表。 显式地或以整个子网（例如 216.58.195.78/28）的形式允许多个 IP 地址。
* 工作区管理员可以选择性地指定要阻止的 IP 地址或子网，即使它们包含在允许列表中也是如此。 这就是所谓的阻止列表。 如果允许的 IP 地址范围包括较小范围的基础结构 IP 地址（这些地址实际上超出了实际的安全网络外围），则可以使用此功能。
* 工作区管理员使用 REST API 来更新允许和阻止的 IP 地址和子网的列表。

## <a name="feature-details"></a><a id="details"> </a><a id="feature-details"> </a>功能详细信息

利用 IP 访问列表 API，Azure Databricks 管理员可以为工作区配置 IP 允许列表和阻止列表。 如果对工作区禁用了该功能，则允许所有访问。 支持允许列表（包含）和阻止列表（排除）。

尝试连接时：

1. 首先，检查所有阻止列表。 如果连接 IP 地址与任何阻止列表匹配，则连接将被拒绝。

2. 如果连接未被阻止列表拒绝，则 IP 地址将与允许列表进行比较。 如果工作区至少有一个允许列表，则仅当 IP 地址与一个允许列表匹配时，才允许连接。 如果工作区没有允许列表，则允许所有 IP 地址。

对于所有合并的允许列表和阻止列表，工作区最多支持 1000 个 IP/CIDR 值，其中一个 CIDR 作为单个值计数。

更改 IP 访问列表功能后，可能需要几分钟更改才能生效。

> [!div class="mx-imgBorder"]
> ![IP 访问列表流关系图](../../_static/images/security/network/ip-access-lists-flow.png)

## <a name="how-to-use-the-ip-access-list-api"></a>如何使用 IP 访问列表 API

本文介绍可以通过该 API 执行的最常见任务。 有关完整的 REST API 参考，请下载 [OpenAPI 规范](../../_static/api-refs/ip-access-list-azure.yaml)并直接或使用读取 OpenAPI 3.0 的应用程序查看它。

若要了解对 Azure Databricks API 的身份验证，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../../dev-tools/api/latest/authentication.md)。

本文中所述的终结点的基路径为 `https://<databricks-instance>/api/2.0`，其中 `<databricks-instance>` 是 Azure Databricks 部署的 `adb-<workspace-id>.<random-number>.databricks.azure.cn` 域名。

## <a name="check-if-your-workspace-has-the-ip-access-list-feature-enabled"></a>检查工作区是否启用了 IP 访问列表功能

若要检查工作区是否启用了 IP 访问列表功能，请调用[获取功能状态 API](../../_static/api-refs/ip-access-list-azure.yaml) (`GET /workspace-conf`)。 将 `keys=enableIpAccessLists` 作为参数传递到请求。

在响应中，`enableIpAccessLists` 字段指定 `true` 或 `false`。

例如：

```bash
curl -X -n \
 https://<databricks-instance>/api/2.0/workspace-conf?keys=enableIpAccessLists
```

示例响应:

```json
{
  "enableIpAccessLists": "true",
}
```

## <a name="enable-or-disable-the-ip-access-list-feature-for-a-workspace"></a>启用或禁用工作区的 IP 访问列表功能

若要启用或禁用工作区的 IP 访问列表功能，请调用[启用或禁用 IP 访问列表 API](../../_static/api-refs/ip-access-list-azure.yaml) (`PATCH /workspace-conf`)。

在 JSON 请求正文中，将 `enableIpAccessLists` 指定为 `true`（已启用）或 `false`（已禁用）。

例如，启用该功能：

```bash
curl -X PATCH -n \
  https://<databricks-instance>/api/2.0/workspace-conf \
  -d '{
    "enableIpAccessLists": "true"
    }'
```

示例响应:

```json
{
  "enableIpAccessLists": "true"
}
```

## <a name="add-an-ip-access-list"></a>添加 IP 访问列表

若要添加 IP 访问列表，请调用[添加 IP 访问列表 API](../../_static/api-refs/ip-access-list-azure.yaml) (`POST /ip-access-lists`)。

在 JSON 请求正文中，指定：

* `label` - 此列表的标签。
* `list_type` - `ALLOW`（允许列表）或 `BLOCK`（阻止列表，这意味着即使在允许列表中也要排除）。
* `ip_addresses` - 一个 IP 地址和 CIDR 范围的 JSON 数组，作为字符串值。

响应是你传入的对象的副本，但带有一些其他字段，最重要的是 `list_id` 字段。 你可能希望保存该值，以便之后可以更新或删除列表。 如果你不保存它，稍后仍可以通过对 `/ip-access-lists` 终结点的 `GET` 请求来查询完整的 IP 访问列表集，从而获取该 ID。

例如，添加允许列表：

```bash
curl -X POST -n \
  https://<databricks-instance>/api/2.0/ip-access-lists
  -d '{
    "label": "office",
    "list_type": "ALLOW",
    "ip_addresses": [
        "1.1.1.1",
        "2.2.2.2/21"
      ]
    }'
```

示例响应:

```json
{
  "ip_access_list": {
    "list_id": "<list-id>",
    "label": "office",
    "ip_addresses": [
        "1.1.1.1",
        "2.2.2.2/21"
    ],
    "address_count": 2,
    "list_type": "ALLOW",
    "created_at": 1578423494457,
    "created_by": 6476783916686816,
    "updated_at": 1578423494457,
    "updated_by": 6476783916686816,
    "enabled": true
  }
}
```

若要添加阻止列表，请执行相同的操作，但请将 `list_type` 设置为 `BLOCK`。

## <a name="update-an-ip-access-list"></a>更新 IP 访问列表

若要更新 IP 访问列表，请调用[更新 IP 访问列表 API](../../_static/api-refs/ip-access-list-azure.yaml) (`PUT /ip-access-lists/<list-id>`)。

在 JSON 请求正文中，指定：

* `label` - 此列表的标签。
* `list_type` - `ALLOW`（允许列表）或 `BLOCK`（阻止列表，这意味着即使在允许列表中也要排除）。
* `ip_addresses` - 一个 IP 地址和 CIDR 范围的 JSON 数组，作为字符串值。
* `enabled` - 指定是否启用了该列表。 传递 `true` 或 `false`。

响应是你传入的对象的副本，其中包含其他的 ID 和修改日期字段。

例如，更新允许列表以禁用它：

```bash
curl -X PUT -n \
  https://<databricks-instance>/api/2.0/ip-access-lists/<list-id>
  -d '{
    "label": "office",
    "list_type": "ALLOW",
    "ip_addresses": [
        "1.1.1.1",
        "2.2.2.2/21"
      ],
    "enabled": "false"
    }'
```

## <a name="delete-an-ip-access-list"></a>删除 IP 访问列表

若要删除 IP 访问列表，请调用[删除 IP 访问列表 API](../../_static/api-refs/ip-access-list-azure.yaml) (`DELETE /ip-access-lists/<list-id>`)。

```bash
curl -X DELETE -n \
  https://<databricks-instance>/api/2.0/ip-access-lists/<list-id>
  -d '{
    "label": "office",
    "list_type": "ALLOW",
    "ip_addresses": [
        "1.1.1.1",
        "2.2.2.2/21"
      ],
    "enabled": "false"
    }'
```