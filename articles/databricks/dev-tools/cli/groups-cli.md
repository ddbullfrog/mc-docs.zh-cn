---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 组 CLI - Azure Databricks
description: 了解如何使用 Databricks 组命令行界面。
ms.openlocfilehash: 943d3f7916987e526c0eb996b530f5122d6c0e5f
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937640"
---
# <a name="groups-cli"></a>组 CLI

> [!NOTE]
>
> * 组 CLI 需要 Databricks CLI 0.8.0 或更高版本。
> * 必须是 Databricks [管理员](../../administration-guide/users-groups/users.md)才能调用此 API。

可以通过将 Databricks 组 CLI 子命令追加到 `databricks groups` 后面来运行这些命令。

```bash
databricks groups --help
```

```
Usage: databricks groups [OPTIONS] COMMAND [ARGS]...

  Provide utility to interact with Databricks groups.

Options:
  -v, --version   0.8.0
  --debug         Debug Mode. Shows full stack trace on error.
  --profile TEXT  CLI connection profile to use. The default profile is "DEFAULT".
  -h, --help      Show this message and exit.

Commands:
  add-member     Add an existing principal to another existing group.
    Options:
      --parent-name TEXT  Name of the parent group to which the new member will be
                          added. This field is required.  [required]
      --user-name TEXT    The user name which will be added to the parent group.
      --group-name TEXT   If group name which will be added to the parent group.
  create         Create a new group with the given name.
    Options:
      --group-name TEXT  [required]
  delete         Remove a group from this organization.
    Options:
      --group-name TEXT  [required]
  list           Return all of the groups in a workspace.
  list-members   Return all of the members of a particular group.
    Options:
      --group-name TEXT  [required]
  list-parents   Retrieve all groups in which a given user or group is a member.
    Options:
      --user-name TEXT
      --group-name TEXT
  remove-member  Removes a user or group from a group.
    Options:
      --parent-name TEXT  Name of the parent group to which the new member will be
                          removed. This field is required.  [required]
      --user-name TEXT    The user name which will be removed from the parent
                          group.
      --group-name TEXT   If group name which will be removed from the parent
                          group.
```

## <a name="list-groups"></a>列出组

```bash
databricks groups list
```

```json
{
  "group_names": [
    "admins"
  ]
}
```

## <a name="list-the-members-of-admins"></a>列出 `admins` 的成员

```bash
databricks groups list-members --group-name admins
```

```json
{
  "members": [
    {
      "user_name": "adminA@example.com"
    },
    {
      "user_name": "adminB@example.com"
    }
  ]
}
```

## <a name="add-group-finance"></a>添加 `finance` 组

```bash
databricks groups create --group-name finance
```

```json
{
  "group_name": "finance"
}
```

```bash
databricks groups list
```

```json
{
  "group_names": [
    "admins",
    "finance"
  ]
}
```