---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: 群集策略 API - Azure Databricks
description: 了解如何根据一组预定义的规则，定义和使用策略来限制用户和用户组的群集创建功能。
ms.openlocfilehash: 59d89cd418ed36e4d1eb2fcc2f085f69495c052e
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937658"
---
# <a name="cluster-policies-apis"></a>群集策略 API

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

群集策略会限制基于一组规则创建群集的功能。
策略规则会限制可用于创建群集的属性或属性值。
群集策略具有将策略的使用限制到特定用户和组的 ACL。

只有管理员用户才能创建、编辑和删除策略。 管理员用户也有权访问所有策略。

有关群集策略的要求和限制，请参阅[管理群集策略](../../../administration-guide/clusters/policies.md)。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="cluster-policies-api"></a>群集策略 API

通过群集策略 API 可创建、列出和编辑群集策略。
只有管理员可以创建和编辑群及策略。 可以由任何用户执行列表操作，并且仅限于列出该用户可访问的策略。

> [!IMPORTANT]
>
> 群集策略 API 要求在 JSON 请求中以字符串化形式传递[策略 JSON 定义](../../../administration-guide/clusters/policies.md#policy-def)。 在大多数情况下，这需要转义引号字符。

### <a name="in-this-section"></a>本节内容：

* [Get](#get)
* [列表](#list)
* [创建](#create)
* [编辑](#edit)
* [删除](#delete)
* [数据结构](#data-structures)

### <a name="get"></a><a id="clusterpolicyservicegetpolicy"> </a><a id="get"> </a>获取

| 端点                          | HTTP 方法     |
|-----------------------------------|-----------------|
| `2.0/policies/clusters/get`       | `GET`           |

返回给定策略 ID 的策略规范。

#### <a name="request-structure"></a><a id="clustergetpolicy"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                            |
|----------------|----------------|--------------------------------------------------------|
| policy_id      | `STRING`       | 要检索其信息的策略 ID。     |

#### <a name="response-structure"></a><a id="clustergetpolicyresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称               | 类型           | 描述                                                                                                                                                                     |
|--------------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| policy_id                | `STRING`       | 群集策略的规范唯一标识符。                                                                                                                             |
| name                     | `STRING`       | 群集策略名称。 名称必须是唯一的。 长度必须介于 1 到 100 个字符之间。                                                                                          |
| 定义               | `STRING`       | 用 Databricks 策略定义语言表示的策略定义 JSON 文档。 JSON 文档必须作为字符串传递，不能简单地嵌入到请求中。 |
| created_at_timestamp     | `INT64`        | 创建时间。 创建此群集策略时的时间戳（毫秒）。                                                                                             |

### <a name="list"></a><a id="clusterpolicyservicelistpolicies"> </a><a id="list"> </a>列表

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/policies/clusters/list`       | `GET`           |

返回请求用户可访问的策略的列表。

#### <a name="request-structure"></a><a id="clusterlistpolicies"> </a><a id="request-structure"> </a>请求结构

| 字段名称      | 类型                                                      | 描述                                                                                        |
|-----------------|-----------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| sort_order      | [ListOrder](clusters.md#clusterlistorder)                 | 列出策略的顺序方向；`ASC` 或 `DESC`。 默认为 `DESC`。           |
| sort_column     | [PolicySortColumn](#clusterpolicysortcolumn)              | 排序依据的 `ClusterPolicy` 属性。 默认为 `CREATION_TIME`。                             |

#### <a name="response-structure"></a><a id="clusterlistpoliciesresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称      | 类型                                 | 描述                       |
|-----------------|--------------------------------------|-----------------------------------|
| 策略        | [策略](#clusterpolicy)的数组 | 策略列表。                 |
| total_count     | `INT64`                              | 策略总数。     |

##### <a name="example"></a>示例

```json
{
  "policies": [
    {
      "policy_id": "ABCD000000000000",
      "name": "Test policy",
      "definition": "{\"spark_conf.spark.databricks.cluster.profile\":{\"type\":\"forbidden\",\"hidden\":true}}",
      "created_at_timestamp": 1600000000000
    }
    {
      "policy_id": "ABCD000000000001",
      "name": "Empty",
      "definition": "{}",
      "created_at_timestamp": 1600000000002
    }
  ],
  "total_count": 1
}
```

### <a name="create"></a><a id="clusterpolicyservicecreatepolicy"> </a><a id="create"> </a>创建

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/policies/clusters/create`       | `POST`          |

使用给定的名称和定义创建新策略。

#### <a name="request-structure"></a><a id="clustercreatepolicy"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 说明                                                                                                                                                                      |
|----------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name           | `STRING`       | 群集策略名称。 名称必须是唯一的。 长度必须介于 1 到 100 个字符之间。                                                                                           |
| 定义     | `STRING`       | 用 Databricks 策略定义语言表示的策略定义 JSON 文档。 必须将 JSON 文档作为字符串传递；不能简单地嵌入到请求中。 |

##### <a name="example"></a>示例

```json
{
  "name": "Example Policy",
  "definition": "{\"spark_version\":{\"type\":\"fixed\",\"value\":\"next-major-version-scala2.12\",\"hidden\":true}}"
}
```

#### <a name="response-structure"></a><a id="clustercreatepolicyresponse"> </a><a id="response-structure"> </a>响应结构

| 字段名称               | 类型           | 描述                                             |
|--------------------------|----------------|---------------------------------------------------------|
| policy_id                | `STRING`       | 群集策略的规范唯一标识符。     |

##### <a name="example"></a>示例

```json
{
  "policy_id": "ABCD000000000000",
}
```

### <a name="edit"></a><a id="clusterpolicyserviceeditpolicy"> </a><a id="edit"> </a>编辑

| 端点                           | HTTP 方法     |
|------------------------------------|-----------------|
| `2.0/policies/clusters/edit`       | `POST`          |

更新现有策略。 这可能会使此策略控制的某些群集无效。
对于此类群集，下一次群集编辑必须提供确认配置，但即使不这么也能继续运行。

#### <a name="request-structure"></a><a id="clustereditpolicy"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                                                                                                      |
|----------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| policy_id      | `STRING`       | 要更新的策略的 ID。 此字段为必需字段。                                                                                                                          |
| name           | `STRING`       | 群集策略名称。 名称必须是唯一的。 长度必须介于 1 到 100 个字符之间。                                                                                           |
| 定义     | `STRING`       | 用 Databricks 策略定义语言表示的策略定义 JSON 文档。 必须将 JSON 文档作为字符串传递；不能简单地嵌入到请求中。 |

##### <a name="example"></a>示例

```json
{
  "policy_id": "ABCD000000000000",
  "name": "Example Policy",
  "definition": "{\"spark_version\":{\"type\":\"fixed\",\"value\":\"next-major-version-scala2.12\",\"hidden\":true}}"
}
```

### <a name="delete"></a><a id="clusterpolicyservicedeletepolicy"> </a><a id="delete"> </a>删除

| 端点                             | HTTP 方法     |
|--------------------------------------|-----------------|
| `2.0/policies/clusters/delete`       | `POST`          |

删除策略。 此策略控制的群集仍可运行，但无法编辑。

#### <a name="request-structure"></a><a id="clusterdeletepolicy"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                             |
|----------------|----------------|---------------------------------------------------------|
| policy_id      | `STRING`       | 要删除的策略的 ID。 此字段为必需字段。 |

##### <a name="example"></a>示例

```json
{
  "policy_id": "ABCD000000000000"
}
```

### <a name="data-structures"></a><a id="data-structures"> </a><a id="restadd"> </a>数据结构

#### <a name="in-this-section"></a>本节内容：

* [策略](#policy)
* [PolicySortColumn](#policysortcolumn)

#### <a name="policy"></a><a id="clusterpolicy"> </a><a id="policy"> </a>策略

群集策略实体。

| 字段名称               | 类型           | 描述                                                                                                                                                                      |
|--------------------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| policy_id                | `STRING`       | 群集策略的规范唯一标识符。                                                                                                                              |
| name                     | `STRING`       | 群集策略名称。 名称必须是唯一的。 长度必须介于 1 到 100 个字符之间。                                                                                           |
| 定义               | `STRING`       | 用 Databricks 策略定义语言表示的策略定义 JSON 文档。 必须将 JSON 文档作为字符串传递；不能简单地嵌入到请求中。 |
| creator_user_name        | `STRING`       | 创建者用户名。 如果已删除用户，则该字段不会包含在响应中。                                                                             |
| created_at_timestamp     | `INT64`        | 创建时间。 创建此群集策略时的时间戳（毫秒）。                                                                                              |

#### <a name="policysortcolumn"></a><a id="clusterpolicysortcolumn"> </a><a id="policysortcolumn"> </a>PolicySortColumn

`ListPolices` 请求的排序顺序。

| 名称                     | 描述                                   |
|--------------------------|-----------------------------------------------|
| POLICY_CREATION_TIME     | 按策略创建类型对结果列表进行排序。     |
| POLICY_NAME              | 按策略名称对结果列表进行排序。              |

## <a name="cluster-policy-permissions-api"></a>群集策略权限 API

使用群集策略权限 API，可以设置群集策略的权限。 向用户授予策略的 `CAN_USE` 权限时，该用户将能够基于该权限创建新群集。 用户不需要 `cluster_create` 权限来创建新的群集。

只有管理员用户可以设置群集策略的权限。

在以下终结点中，`<basepath>` = `/api/2.0/preview`。

### <a name="in-this-section"></a>本节内容：

* [获取权限](#get-permissions)
* [添加或修改权限](#add-or-modify-permissions)
* [设置或删除权限](#set-or-delete-permissions)
* [数据结构](#data-structures)

### <a name="get-permissions"></a><a id="clustergetpermissions"> </a><a id="get-permissions"> </a>获取权限

| 端点                                                                           | HTTP 方法     |
|------------------------------------------------------------------------------------|-----------------|
| `<basepath>/permissions/cluster-policies/<clusterPolicyId>/permissionLevels`       | `GET`           |

#### <a name="request-structure"></a><a id="clustergetpolicygetrequest"> </a><a id="request-structure"> </a>请求结构

| 字段名称          | 类型           | 描述                                                             |
|---------------------|----------------|-------------------------------------------------------------------------|
| clusterPolicyId     | `STRING`       | 要检索其权限的策略。 此字段为必需字段。 |

#### <a name="response-structure"></a><a id="clustergetpolicygetresponse"> </a><a id="response-structure"> </a>响应结构

[群集 ACL](#clustersacl)。

#### <a name="example-response"></a>示例响应

```json
{
  "object_id": "/cluster-policies/D55CAFDD8E00002B",
  "object_type": "cluster-policy",
  "access_control_list": [
    {
      "user_name": "user@mydomain.com",
      "all_permissions": [
        {
          "permission_level": "CAN_USE",
          "inherited": false
        },
      ]
    },
    {
      "group_name": "admins",
      "all_permissions": [
        {
          "permission_level": "CAN_USE",
          "inherited": true,
          "inherited_from_object": [
              "/cluster-policies/"
          ]
        }
      ]
    },
  ]
}
```

### <a name="add-or-modify-permissions"></a>添加或修改权限

| 端点                                                                           | HTTP 方法     |
|------------------------------------------------------------------------------------|-----------------|
| `<basepath>/permissions/cluster-policies/<clusterPolicyId>`                        | `PATCH`         |

#### <a name="request-structure"></a><a id="clustergetpolicymodifyrequest"> </a><a id="request-structure"> </a>请求结构

| 字段名称          | 类型           | 描述                                                           |
|---------------------|----------------|-----------------------------------------------------------------------|
| clusterPolicyId     | `STRING`       | 要修改其权限的策略。 此字段为必需字段。 |

#### <a name="request-body"></a><a id="clustergetpolicymodifyresponse"> </a><a id="request-body"> </a>请求正文

| 字段名称                           | 类型                                                 | 描述                              |
|--------------------------------------|------------------------------------------------------|------------------------------------------|
| access_control_list                  | [AccessControl](#clusterpolicyacl) 的数组          | 访问控制列表的数组。        |

#### <a name="response-body"></a>响应正文

与对 `<clusterPolicyId>` 执行的 GET 调用相同，返回已修改的群集权限。

### <a name="set-or-delete-permissions"></a>设置或删除权限

PUT 请求替代群集策略对象上的所有直接权限。 发出删除请求的方法是发出请求检索当前权限列表的 GET 请求，然后发出请求删除所需条目的 PUT 请求。

| 端点                                                                           | HTTP 方法     |
|------------------------------------------------------------------------------------|-----------------|
| `<basepath>/permissions/cluster-policies/<clusterPolicyId>`                        | `PUT`           |

#### <a name="request-structure"></a><a id="clustergetpolicysetrequest"> </a><a id="request-structure"> </a>请求结构

| 字段名称          | 类型           | 描述                                                        |
|---------------------|----------------|--------------------------------------------------------------------|
| clusterPolicyId     | `STRING`       | 要设置其权限的策略。 此字段为必需字段。 |

#### <a name="request-body"></a>请求正文

| 字段名称                           | 类型                                                 | 描述                              |
|--------------------------------------|------------------------------------------------------|------------------------------------------|
| access_control_list                  | [AccessControlInput](#clusterpolicyaclitem) 的数组 | 访问控制的数组。             |

#### <a name="response-body"></a><a id="clustergetpolicysetresponse"> </a><a id="response-body"> </a>响应正文

与对 `<clusterPolicyId>` 执行的 GET 调用相同，返回已修改的群集权限。

### <a name="data-structures"></a>数据结构

#### <a name="in-this-section"></a>本节内容：

* [群集 ACL](#clusters-acl)
* [AccessControl](#accesscontrol)
* [权限](#permission)
* [AccessControlInput](#accesscontrolinput)
* [PermissionLevel](#permissionlevel)

#### <a name="clusters-acl"></a><a id="clusters-acl"> </a><a id="clustersacl"> </a>群集 ACL

| 属性名称           | 类型                                        | 描述                                                                                  |
|--------------------------|---------------------------------------------|----------------------------------------------------------------------------------------------|
| object_id                | STRING                                      | ACL 对象的 ID，例如 `../cluster-policies/<clusterPolicyId>`。              |
| object_type              | STRING                                      | Databricks ACL 对象类型，例如 `cluster-policy`。                               |
| access_control_list      | [AccessControl](#clusterpolicyacl) 的数组 | ACL 对象上设置的访问控制。                                                   |

#### <a name="accesscontrol"></a><a id="accesscontrol"> </a><a id="clusterpolicyacl"> </a>AccessControl

| 属性名称                   | 类型                                            | 描述                                                                                                                                                                               |
|----------------------------------|-------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| user_name 或 group_name          | STRING                                          | 在 ACL 对象上设置了权限的主体（用户或组）的名称。                                                                                                         |
| all_permissions                  | [权限](#clusterpolicypermission)的数组 | 在此 ACL 对象上为特定主体设置的所有权限的列表。 既包括直接在此 ACL 对象上设置的权限，也包括从上级 ACL 对象继承的权限。 |

#### <a name="permission"></a><a id="clusterpolicypermission"> </a><a id="permission"> </a>权限

| 属性名称            | 类型             | 描述                                                                                                                            |
|---------------------------|------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| permission_level          | STRING           | 权限级别的名称。                                                                                                      |
| 已继承                 | BOOLEAN          | 如果不是直接设置 ACL 权限而是继承自上级 ACL 对象，则为 True。 如果是直接在 ACL 对象上设置，则为 False。   |
| inherited_from_object     | List[STRING]     | ACL 对象的继承权限所涉及的父 ACL 对象 ID 列表。 仅当继承为 true 时才定义此值。 |

#### <a name="accesscontrolinput"></a><a id="accesscontrolinput"> </a><a id="clusterpolicyaclitem"> </a>AccessControlInput

代表应用于主体（用户或组）的 ACL 规则的项。

| 属性名称                   | 类型      | 描述                                                                         |
|----------------------------------|-----------|-------------------------------------------------------------------------------------|
| user_name 或 group_name          | STRING    | 在 ACL 对象上设置了权限的主体（用户或组）的名称。   |
| permission_level                 | STRING    | 权限级别的名称。                                                   |

#### <a name="permissionlevel"></a>PermissionLevel

可以在群集策略上设置的权限级别。

| 权限级别        | 描述                                                                                              |
|-------------------------|----------------------------------------------------------------------------------------------------------|
| CAN_USE                 | 允许用户基于策略创建群集。 用户无需群集创建权限。 |