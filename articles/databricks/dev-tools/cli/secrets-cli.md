---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 机密 CLI - Azure Databricks
description: 了解如何使用 Databricks 机密命令行界面。
ms.openlocfilehash: 3208d3e4eaf138904cd573a02473d0fc758e5d24
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937637"
---
# <a name="secrets-cli"></a>机密 CLI

> [!NOTE]
>
> 机密 CLI 需要 Databricks CLI 0.7.1 或更高版本。

可以通过将 Databricks 机密 CLI 子命令附加到 `databricks secrets` 来运行这些命令。

有关机密的详细信息，请参阅[机密管理](../../security/secrets/index.md#secrets-user-guide)。

```bash
databricks secrets --help
```

```
Usage: databricks secrets [OPTIONS] COMMAND [ARGS]...

  Utility to interact with secret API.

Options:
  -v, --version   [VERSION]
  --profile TEXT  CLI connection profile to use. The default profile is
                  "DEFAULT".
  -h, --help      Show this message and exit.

Commands:
  create-scope  Creates a secret scope.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --initial-manage-principal     The initial principal that can manage the created secret scope.
                                      If specified, the initial ACL with MANAGE permission applied
                                      to the scope is assigned to the supplied principal (user or group).
                                      The only supported principal is the group
                                      "users", which contains all users in the workspace. If not
                                      specified, the initial ACL with MANAGE permission applied to
                                      the scope is assigned to request issuer's user identity.
  delete        Deletes a secret.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --key KEY                      The name of secret key.
  delete-acl    Deletes an access control rule for a principal.
    Options:
      --scope SCOPE                  The name of the scope.
      --principal PRINCIPAL          The name of the principal.
  delete-scope  Deletes a secret scope.
    Options:
      --scope SCOPE                  The name of the secret scope.
  get-acl       Gets the details for an access control rule.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --principal PRINCIPAL          The name of the principal.
      --output FORMAT                JSON or TABLE. Set to TABLE by default.
  list          Lists all the secrets in a scope.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --output FORMAT                JSON or TABLE. Set to TABLE by default.
  list-acls     Lists all access control rules for a given secret scope.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --output FORMAT                JSON or TABLE. Set to TABLE by default.
  list-scopes   Lists all secret scopes.
      --output FORMAT                JSON or TABLE. Set to TABLE by default.
  put           Puts a secret in a scope.
    Options:
      --scope SCOPE                  The name of the secret scope.
      --key KEY                      The name of the secret key.  [required]
      --string-value TEXT            Read value from string and stored in UTF-8 (MB4) form
      --binary-file PATH             Read value from binary-file and stored as bytes.
  put-acl       Creates or overwrites an access control rule for a principal
                applied to a given secret scope.
    Options:
      --scope SCOPE                    The name of the secret scope.
      --principal PRINCIPAL            The name of the principal.
      --permission [MANAGE|WRITE|READ] The permission to apply.
```

## <a name="create-a-secret-scope"></a>创建机密范围

```bash
databricks secrets create-scope --scope my-scope
```

## <a name="list-all-secret-scopes-in-workspace"></a>列出工作区中的所有机密范围

```bash
databricks secrets list-scopes
```

## <a name="delete-a-secret-scope"></a>删除机密范围

```bash
databricks secrets delete-scope --scope my-scope
```

## <a name="create-or-update-a-secret-in-a-secret-scope"></a>在机密范围内创建或更新机密

可通过三种方式存储机密。 最简单的方法是使用 `--string-value` 选项；机密将以 UTF-8 (MB4) 格式存储。 你应谨慎使用此选项，因为你的机密可能以纯文本形式存储在你的命令行历史记录中。

```bash
databricks secrets put --scope my-scope --key my-key --string-value my-value
```

你还可以使用 `--binary-file` 选项提供存储在文件中的机密。 将按原样读取文件内容并将其以字节形式存储。

```bash
databricks secrets put --scope my-scope --key my-key --binary-file my-secret.txt
```

如果你未指定这两个选项中的任何一个，系统会打开编辑器供你输入机密。 请按照编辑器上显示的说明输入机密。

```bash
databricks secrets put --scope my-scope --key my-key
```

## <a name="list-secrets-stored-within-the-secret-scope"></a>列出机密范围内存储的机密

```bash
databricks secrets list --scope my-scope
```

没有用于从 CLI 获取机密的界面。 必须使用 Databricks 笔记本中的 [Databricks 实用工具](../databricks-utils.md)机密实用工具界面来访问机密。

## <a name="delete-a-secret-in-a-secret-scope"></a>删除机密范围内的机密

```bash
databricks secrets delete --scope my-scope --key my-key
```

## <a name="grant-or-change-acl-for-a-principal"></a>授予或更改主体的 ACL

```bash
databricks secrets put-acl --scope my-scope --principal principal --permission MANAGE
```

## <a name="list-acls-in-a-secret-scope"></a>列出机密范围内的 ACL

```bash
databricks secrets list-acls --scope my-scope
```

## <a name="get-acl-for-a-principal-in-a-secret-scope"></a>获取机密范围内主体的 ACL

```bash
databricks secrets get-acl --scope my-scope --principal principal
```

## <a name="revoke-acl-for-a-principal"></a>撤销主体的 ACL

```bash
databricks secrets delete-acl --scope my-scope --principal principal
```