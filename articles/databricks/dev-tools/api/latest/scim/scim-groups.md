---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: SCIM API（组）- Azure Databricks
description: 了解如何使用 Databricks SCIM API 在 Azure Databricks 中创建、读取、更新和删除组。
ms.openlocfilehash: 36fbe3112b9f4a8c5489b7aaa8cd4f114ef1a3a2
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937655"
---
# <a name="scim-api-groups"></a>SCIM API（组）

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../../release-notes/release-types.md)提供。

> [!NOTE]
>
> * Azure Databricks [管理员](../../../../administration-guide/index.md#administration)可以调用所有 SCIM API 终结点。
> * 非管理员用户可以调用“获取组”终结点以读取组显示名称和 ID。

借助 SCIM（组），可以在 Azure Databricks 中创建用户和组并为他们提供适当的访问级别；当他们离开你的组织或不再需要访问 Azure Databricks 时，你还可以删除他们的访问权限（将他们取消预配）。

## <a name="get-groups"></a>获取组

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Groups`                 | `GET`           |

管理员用户：在 Azure Databricks 工作区中检索所有组的列表。
非管理员用户：检索 Azure Databricks 工作区中所有组的列表，仅返回组显示名称和对象 ID。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/Groups  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

可以使用[筛选器](index.md#filter-results)来指定组的子集。 例如，可以将 `sw`（“开头为”）筛选器参数应用于 `displayName` 以检索特定组或组集：

```http
GET /api/2.0/preview/scim/v2/Groups?filter=displayName+sw+eng    HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

## <a name="get-group-by-id"></a>按 ID 获取组

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Groups/{id}`            | `GET`           |

管理员用户：检索单个组资源。

### <a name="example-request"></a>示例请求

```http
GET /api/2.0/preview/scim/v2/Groups/123456  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```

## <a name="create-group"></a>创建组

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Groups`                 | `POST`          |

管理员用户：在 Azure Databricks 中创建组。

请求参数遵循标准 SCIM 2.0 协议。

请求必须包括以下属性：

* 将 `schemas` 设置为 `urn:ietf:params:scim:schemas:core:2.0:Group`
* `displayName`

`Members` 列表是可选项，可以包含用户和其他组。  还可以使用 `PATCH` 向组添加成员。

### <a name="example-request"></a>示例请求

```http
POST /api/2.0/preview/scim/v2/Groups HTTP/1.1
Host: <databricks-instance>
Authorization: Bearer dapi48…a6138b
Content-Type: application/scim+json
```

```json
{
  "schemas":[
    "urn:ietf:params:scim:schemas:core:2.0:Group"
  ],
  "displayName":"newgroup",
  "members":[
    {
       "value":"100000"
    },
    {
       "value":"100001"
    }
  ]
}
```

## <a name="update-group"></a>更新组

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Groups/{id}`            | `PATCH`         |

管理员用户：通过添加或删除成员，更新 Azure Databricks 中的组。 可以在组中添加和删除单个成员或组。

请求参数遵循标准 SCIM 2.0 协议，并依赖于 `schemas` 属性的值。

> [!NOTE]
>
> Azure Databricks 不支持更新组名。

### <a name="example-requests"></a>示例请求

```http
PATCH /api/2.0/preview/scim/v2/Groups/123456 HTTP/1.1
Host: <databricks-instance>
Authorization: Bearer dapi48…a6138b
Content-Type: application/scim+json
```

#### <a name="add-to-group"></a>添加到组

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
    "op":"add",
    "value":{
        "members":[
           {
              "value":"<user-id>"
           }
        ]
      }
    }
  ]
}
```

#### <a name="remove-from-group"></a>从组中移除

```json
{
  "schemas":[
    "urn:ietf:params:scim:api:messages:2.0:PatchOp"
  ],
  "Operations":[
    {
      "op":"remove",
      "path":"members[value eq \"<user-id>\"]"
    }
  ]
}
```

## <a name="delete-group"></a>删除组

| 端点                                     | HTTP 方法     |
|----------------------------------------------|-----------------|
| `2.0/preview/scim/v2/Groups/{id}`            | `DELETE`        |

管理员用户：从 Azure Databricks 中删除组。 不会删除组中的用户。

### <a name="example-request"></a>示例请求

```http
DELETE /api/preview/scim/v2/Groups/123456  HTTP/1.1
Host: <databricks-instance>
Accept: application/scim+json
Authorization: Bearer dapi48…a6138b
```