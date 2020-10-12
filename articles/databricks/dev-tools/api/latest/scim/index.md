---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: SCIM API - Azure Databricks
description: 了解如何使用 Databricks SCIM API。
ms.openlocfilehash: 75053dc982ac1181fed3d4d69e704aeeb7a0e747
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937656"
---
# <a name="scim-api"></a>SCIM API

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

Azure Databricks 支持 [SCIM](http://www.simplecloud.info/)（跨域身份管理系统，一种可用于使用 REST API 和 JSON 将用户预配过程自动化的开放标准）。 Azure Databricks SCIM API 遵循 SCIM 协议版本 2.0。

> [!NOTE]
>
> * Azure Databricks [管理员](../../../../administration-guide/users-groups/users.md)可以调用所有 SCIM API 终结点。
> * 非管理员用户可以调用“获取[我](scim-me.md)”终结点、“获取[用户](scim-users.md)”终结点以读取用户显示名称和 ID，并可以调用“获取[组](scim-groups.md)”终结点以读取组显示名称和 ID。

## <a name="call-the-scim-api"></a>调用 SCIM API

在示例中，请将 `<databricks-instance>` 替换为 Azure Databricks 部署的[工作区 URL](../../../../workspace/workspace-details.md#workspace-url)。

### <a name="resource-url"></a>资源 URL

```
https://<databricks-instance>/api/2.0/preview/scim/v2/<api-endpoint>
```

### <a name="header-parameters"></a>标头参数

| 参数                                    | 类型           | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|----------------------------------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Authorization（必需）                     | `STRING`       | 设置为 `Bearer <access-token>`。<br><br>若要了解如何生成令牌，请参阅[使用 Azure Databricks 个人访问令牌进行身份验证](../authentication.md)、[使用 Azure Active Directory 令牌进行身份验证](../aad/index.md)和[令牌 API](../tokens.md)。<br><br>**重要说明：** 生成此令牌的 Azure Databricks 管理员用户不应由标识提供者 (IdP) 管理。 可以使用 IdP 来取消预配由 IdP 管理的 Azure Databricks 管理员用户，这会导致 SCIM 预配集成被禁用。 |
| Content-Type（写入操作所必需） | `STRING`       | 设置为 `application/scim+json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Accept（读取操作所必需）        | `STRING`       | 设置为 `application/scim+json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |

## <a name="filter-results"></a>筛选结果

使用筛选器可返回用户或组的子集。 对于所有用户，支持用户 `userName` 和组 `displayName` 字段。 管理员用户可根据 `active` 属性筛选用户。

| 运算符     | 说明                     | 行为                                                        |
|--------------|---------------------------------|-----------------------------------------------------------------|
| eq           | equals                          | 属性值和运算符值必须相同。                |
| ne           | 不等于                    | 属性值和运算符值不相同。                |
| co           | contains                        | 运算符值必须是属性值的子字符串。          |
| sw           | 开头为                     | 属性必须以运算符值开头并包含运算符值。           |
| and          | 逻辑与                     | 当所有表达式的计算结果都为 true 时匹配。                    |
| 或           | 逻辑或                      | 当任意表达式的计算结果为 true 时匹配。                    |

## <a name="sort-results"></a>对结果进行排序

使用 `sortBy` 和 `sortOrder` [查询参数](https://tools.ietf.org/html/rfc7644#section-3.4.2.3)对结果进行排序。 默认选择“按 ID 排序”。

## <a name="apis"></a>API

* [SCIM API（我）](scim-me.md)
* [SCIM API（用户）](scim-users.md)
* [SCIM API（组）](scim-groups.md)
* [SCIM API (ServicePrincipals)](scim-sp.md)