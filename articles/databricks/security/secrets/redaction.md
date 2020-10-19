---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 04/29/2020
title: 机密编修 - Azure Databricks
description: 了解编修如何保护 Azure Databricks 机密免受意外显示，以及如何确保机密的正确控制。
ms.openlocfilehash: a750064e3e53d140f7b08f6931dd29032fa1a91d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937736"
---
# <a name="secret-redaction"></a>机密编修

将凭据存储为 Azure Databricks 密钥，可以在运行笔记本和作业时轻松保护凭据。
但是，很容易意外将机密打印到标准输出缓冲区，或在变量赋值期间显示该值。

为防止出现此情况，Azure Databricks 会编辑使用 `dbutils.secrets.get()` 读取的机密值。 在笔记本单元格输出中显示时，机密值将替换为 `[REDACTED]`。

> [!WARNING]
>
> 笔记本单元格输出的机密编修仅适用于文本。 因此，机密编修功能不会阻止机密文本的故意转换和任意转换。 若要确保机密的正确控制，你应该使用[工作区对象访问控制](../access-control/workspace-acl.md)（限制运行命令的权限），以防止未经授权访问共享笔记本上下文。