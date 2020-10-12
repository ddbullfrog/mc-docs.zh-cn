---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 07/29/2020
title: 表访问控制 - Azure Databricks
description: 了解如何使用表访问控制来允许和拒绝对数据的访问。
ms.openlocfilehash: 77304d204f52538f55655be20f328c54f6cc3451
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937727"
---
# <a name="table-access-control"></a>表访问控制

可以通过表访问控制（表 ACL）以编程方式授予和撤销从 Python 和 SQL 访问数据的权限。

默认情况下，所有用户均有权访问存储在群集的托管表中的所有数据，除非已为该群集启用了表访问控制。 启用表访问控制后，用户可以为该群集上的数据对象设置权限。

## <a name="requirements"></a>要求

访问控制需要 [Azure Databricks Premium 计划](https://databricks.com/product/azure-pricing)。

本部分的内容：

* [为群集启用表访问控制](table-acl.md)
* [数据对象特权](object-privileges.md)