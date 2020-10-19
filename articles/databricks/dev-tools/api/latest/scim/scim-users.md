---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: SCIM API（用户）- Azure Databricks
description: 了解如何使用 Databricks SCIM API 在 Azure Databricks 中创建、读取、更新和删除用户。
ms.openlocfilehash: 969a0e2f623c8117d3f9ad288511bff5f52139ad
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937652"
---
# <a name="scim-api-users"></a>SCIM API（用户）

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> * Azure Databricks [管理员](../../../../administration-guide/index.md#administration)可以调用所有 SCIM API 终结点。
> * 非管理员用户可以调用“获取用户”终结点以读取用户显示名称和 ID。

借助 SCIM（用户），可以在 Azure Databricks 中创建用户并为他们提供适当的访问级别；当他们离开你的组织或不再需要访问 Azure Databricks 时，你还可以删除他们的访问权限（将他们取消预配）。

## <a name="get-users"></a><a id="get-users"> </a><a id="scim-get-users"> </a>获取用户

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users`        | `GET`           |

管理员用户：在 Azure Databricks 工作区中检索所有用户的列表。

非管理员用户：检索 Azure Databricks 工作区中所有用户的列表，仅返回用户名、用户显示名称和对象 ID。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/Users  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

可以使用[筛选器](index.md#filter-results)来指定用户的子集。 例如，可以将 `eq`（“等于”）筛选器参数应用到 `userName` 以检索特定用户或用户子集：

```http
GET /api/2.0/preview/scim/v2/Users?filter=userName+eq+example@databricks.com  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

## <a name="get-user-by-id"></a>按 ID 获取用户

| 端点                               | HTTP 方法     |
|----------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users/{id}`       | `GET`           |

管理员用户：在给定 Azure Databricks ID 的情况下，从 Azure Databricks 工作区中检索单个用户资源。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/Users/100757  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

### <a name="example-response"></a>示例响应

```json
{
  "entitlements":[
    {
      "value":"allow-cluster-create"
    }
  ],
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "groups":[
    {
      "value":"123456"
    }
  ],
  "userName":"example@databricks.com"
}
```

## <a name="create-user"></a>创建用户

| 端点                               | HTTP 方法     |
|----------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users`            | `POST`          |

管理员用户：在 Azure Databricks 工作区中创建用户。

请求参数遵循标准 SCIM 2.0 协议。

请求必须包括以下属性：

* 将 `schemas` 设置为 `urn:ietf:params:scim:schemas:core:2.0:User`
* `userName`

### <a name="example-request"></a>示例请求

```http
POST /api/2.0/preview/scim/v2/Users HTTP/1.1
Host: <databricks-instance>
Authorization: Bearer dapi48…a6138b
Content-Type: application/scim+json
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName":"example@databricks.com",
  "groups":[
    {
       "value":"123456"
    }
  ],
  "entitlements":[
    {
       "value":"allow-cluster-create"
    }
  ]
}
```

#### <a name="powershell-example"></a>PowerShell 示例

```bash
$url = "<databricks-instance>/api/2.0/preview/scim/v2/Users"
$bearer_token = "<token>"
$headers = @{Authorization = "Bearer $bearer_token"}
$par = '{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName":"<username>",
  "displayName":"<firstname lastname>",
  "entitlements":[
    {
    "value":"allow-cluster-create"
    }
  ]
}'

Invoke-WebRequest $url -Method Post -Headers $headers -Body $par -ContentType 'application/json'
```

## <a name="update-user-by-id-patch"></a>按 ID 更新用户 (`PATCH`)

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users/{id}`             | `PATCH`         |

管理员用户：使用对特定属性（不可变属性 `userName` 和 `userId` 除外）的操作来更新用户资源。  建议使用 `PATCH` 方法（而不是 `PUT` 方法）来设置或更新用户权利。

请求参数遵循标准 SCIM 2.0 协议，并依赖于 `schemas` 属性的值。

### <a name="example-request"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/Users/100757  HTTP/1.1
Host: <databricks-instance>
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"add",
      "path":"entitlements",
      "value":[
        {
           "value":"allow-cluster-create"
        }
      ]
    }
  ]
}
```

## <a name="update-user-by-id-put"></a>按 ID 更新用户 (`PUT`)

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users/{id}`             | `PUT`           |

管理员用户：跨多个属性（不可变属性 `userName` 和 `userId` 除外）覆盖用户资源。

请求必须包含设置为 `urn:ietf:params:scim:schemas:core:2.0:User` 的 `schemas` 属性。

> [!NOTE]
>
> 建议使用 `PATCH` 方法（而不是 `PUT` 方法）来设置或更新用户权利。

### <a name="example-request"></a>示例请求

```http
PUT /api/2.0/preview/scim/v2/Users/123456  HTTP/1.1
Host: <region>.databricks.azure.cn
Content-Type: application/scim+json
Authorization: Bearer dapi48…a6138b
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:User"
  ],
  "userName":"example@databricks.com",
  "entitlements":[
    {
       "value":"allow-cluster-create"
    }
  ],
  "groups":[
    {
       "value":"100000"
    }
  ]
}
```

## <a name="delete-user-by-id"></a>按 ID 删除用户

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Users/{id}`             | `DELETE`        |

管理员用户：停用用户资源。  不拥有或不属于 Azure Databricks 工作区的用户将在 30 天后被自动清除。

### <a name="example-request"></a>示例请求

```http
DELETE /api/2.0/preview/scim/v2/Users/100757  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```