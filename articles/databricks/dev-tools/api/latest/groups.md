---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/11/2020
title: Groups API - Azure Databricks
description: 了解 Databricks Groups API。
ms.openlocfilehash: ea60be39188e6fd065e353ddfc6a002add231cf2
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937680"
---
# <a name="groups-api"></a>组 API

通过 Groups API，可以管理用户组。

> [!NOTE]
>
> 必须是 Azure Databricks [管理员](../../../administration-guide/users-groups/users.md)才能调用此 API。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。

## <a name="add-member"></a><a id="aclgroupsserviceaddtogroup"> </a><a id="add-member"> </a>添加成员

| 端点                      | HTTP 方法     |
|-------------------------------|-----------------|
| `2.0/groups/add-member`       | `POST`          |

将用户或组添加到一个组。 如果具有给定名称的用户或组不存在，或者具有给定父名称的组不存在，则此调用将返回错误 `RESOURCE_DOES_NOT_EXIST`。

例如，请求：

```json
{
  "user_name": "hermione@hogwarts.edu",
  "parent_name": "Gryffindor"
}
```

```json
{
  "group_name": "Dumbledore's Army",
  "parent_name": "Students"
}
```

### <a name="request-structure"></a><a id="acladdprincipaltogroup"> </a><a id="request-structure"> </a>请求结构

| 字段名称                          | 类型                         | 描述                                                                             |
|-------------------------------------|------------------------------|-----------------------------------------------------------------------------------------|
| user_name 或 group_name             | `STRING` 或 `STRING`         | 如果 user_name，则用户名。<br><br>如果 group_name，则组名。                      |
| parent_name                         | `STRING`                     | 要向其添加新成员的父组的名称。 此字段为必需字段。 |

## <a name="create"></a><a id="aclgroupsservicecreategroup"> </a><a id="create"> </a>创建

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/groups/create`       | `POST`          |

使用给定名称创建一个新组。 如果具有给定名称的组已经存在，则此调用返回错误 `RESOURCE_ALREADY_EXISTS`。

示例请求：

```json
{
  "group_name": "Muggles"
}
```

示例响应:

```json
{
  "group_name": "Muggles"
}
```

### <a name="request-structure"></a><a id="aclcreategroup"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                                                         |
|----------------|----------------|-----------------------------------------------------------------------------------------------------|
| group_name     | `STRING`       | 组的名称；在该组织拥有的组中必须是唯一的。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="aclcreategroupresponse"> </a><a id="response-structure"> </a>响应结构

创建组。

| 字段名称     | 类型           | 描述         |
|----------------|----------------|---------------------|
| group_name     | `STRING`       | 组名称。     |

## <a name="list-members"></a><a id="aclgroupsservicegetgroupmembers"> </a><a id="list-members"> </a>列出成员

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/groups/list-members`       | `GET`           |

返回特定组的所有成员。 如果具有给定名称的组不存在，则此调用返回错误 `RESOURCE_DOES_NOT_EXIST`。

示例请求：

```json
{
  "group_name": "Gryffindor"
}
```

示例响应:

```json
{
  "members": [
    { "user_name": "hjp@hogwarts.edu" },
    { "user_name": "hermione@hogwarts.edu" },
    { "user_name": "rweasley@hogwarts.edu" },
    { "group_name": "Gryffindor Faculty" }
  ]
}
```

### <a name="request-structure"></a><a id="aclgetgroupmembers"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                                          |
|----------------|----------------|----------------------------------------------------------------------|
| group_name     | `STRING`       | 我们要检索其成员的组。 此字段为必需字段。 |

### <a name="response-structure"></a><a id="aclgetgroupmembersresponse"> </a><a id="response-structure"> </a>响应结构

检索属于给定组的所有用户和组。 此方法是非递归的；它返回属于给定组的所有组，但不返回属于那些子组的主体。

| 字段名称     | 类型                                           | 描述                                              |
|----------------|------------------------------------------------|----------------------------------------------------------|
| members        | [PrincipalName](#aclprincipalname) 的数组 | 属于给定组的用户和组。     |

## <a name="list"></a><a id="aclgroupsservicegetgroups"> </a><a id="list"> </a>列表

| 端点                | HTTP 方法     |
|-------------------------|-----------------|
| `2.0/groups/list`       | `GET`           |

返回组织中的所有组。

示例响应:

```json
{
  "group_names": [
    "admin",
    "Gryffindor",
    "Hufflepuff",
    "Ravenclaw",
    "Slytherin"
  ]
}
```

### <a name="response-structure"></a><a id="aclgetgroupsresponse"> </a><a id="response-structure"> </a>响应结构

返回该组织中的所有组的列表。

| 字段名称      | 类型                       | 描述                          |
|-----------------|----------------------------|--------------------------------------|
| group_names     | 一个由 `STRING` 构成的数组       | 该组织中的组。     |

## <a name="list-parents"></a><a id="aclgroupsservicegetgroupsforprincipal"> </a><a id="list-parents"> </a>列出父级

| 端点                        | HTTP 方法     |
|---------------------------------|-----------------|
| `2.0/groups/list-parents`       | `GET`           |

检索给定用户或组是成员的所有组。 此方法是非递归的；它返回给定用户或组是成员的所有组，但不返回那些组是成员的组。 如果具有给定名称的用户或组不存在，则此调用返回错误 `RESOURCE_DOES_NOT_EXIST`。

示例请求：

```json
{
  "user_name": "hermione@hogwarts.edu"
}
```

示例响应:

```json
{
  "group_names": [
    "users",
    "Wizards",
    "Gryffindor",
    "Dumbledore's Army"
  ]
}
```

示例请求：

```json
{
  "group_name": "Gryffindor Faculty"
}
```

示例响应:

```json
{
  "group_names": [
    "Faculty",
    "Gryffindor"
  ]
}
```

### <a name="request-structure"></a><a id="aclgetgroupsforprincipal"> </a><a id="request-structure"> </a>请求结构

| 字段名称                          | 类型                         | 描述                                                        |
|-------------------------------------|------------------------------|--------------------------------------------------------------------|
| user_name 或 group_name             | `STRING` 或 `STRING`         | 如果 user_name，则用户名。<br><br>如果 group_name，则组名。 |

### <a name="response-structure"></a><a id="aclgetgroupsforprincipalresponse"> </a><a id="response-structure"> </a>响应结构

检索给定用户或组是成员的所有组。 此方法是非递归的；它返回给定用户或组是成员的所有组，但不返回那些组是成员的组。

| 字段名称      | 类型                       | 描述                                                  |
|-----------------|----------------------------|--------------------------------------------------------------|
| group_names     | 一个由 `STRING` 构成的数组       | 给定用户或组是成员的组。     |

## <a name="remove-member"></a><a id="aclgroupsserviceremovefromgroup"> </a><a id="remove-member"> </a>删除成员

| 端点                         | HTTP 方法     |
|----------------------------------|-----------------|
| `2.0/groups/remove-member`       | `POST`          |

从组中删除用户或组。 如果具有给定名称的用户或组不存在，或者具有给定父名称的组不存在，则此调用将返回错误 `RESOURCE_DOES_NOT_EXIST`。

例如，请求：

```json
{
  "user_name": "quirrell@hogwarts.edu",
  "parent_name": "Faculty"
}
```

```json
{
  "group_name": "Inquisitorial Squad",
  "parent_name": "Students"
}
```

### <a name="request-structure"></a><a id="aclremoveprincipalfromgroup"> </a><a id="request-structure"> </a>请求结构

| 字段名称                          | 类型                         | 描述                                                                             |
|-------------------------------------|------------------------------|-----------------------------------------------------------------------------------------|
| user_name 或 group_name             | `STRING` 或 `STRING`         | 如果 user_name，则用户名。<br><br>如果 group_name，则组名。                      |
| parent_name                         | `STRING`                     | 要从中删除成员的父组的名称。 此字段为必需字段。 |

## <a name="delete"></a><a id="aclgroupsserviceremovegroup"> </a><a id="delete"> </a>删除

| 端点                  | HTTP 方法     |
|---------------------------|-----------------|
| `2.0/groups/delete`       | `POST`          |

从该组织中删除一个组。 如果具有给定名称的组不存在，则此调用返回错误 `RESOURCE_DOES_NOT_EXIST`。

示例请求：

```json
{
  "group_name": "Inquisitorial Squad"
}
```

### <a name="request-structure"></a><a id="aclremovegroup"> </a><a id="request-structure"> </a>请求结构

| 字段名称     | 类型           | 描述                                  |
|----------------|----------------|----------------------------------------------|
| group_name     | `STRING`       | 要删除的组。 此字段为必需字段。 |

## <a name="data-structures"></a><a id="data-structures"> </a><a id="groupsadd"> </a>数据结构

### <a name="in-this-section"></a>本节内容：

* [PrincipalName](#principalname)

### <a name="principalname"></a><a id="aclprincipalname"> </a><a id="principalname"> </a>PrincipalName

用户名或组名的容器类型。

| 字段名称                          | 类型                         | 描述                                                        |
|-------------------------------------|------------------------------|--------------------------------------------------------------------|
| user_name 或 group_name             | `STRING` 或 `STRING`         | 如果 user_name，则用户名。<br><br>如果 group_name，则组名。 |