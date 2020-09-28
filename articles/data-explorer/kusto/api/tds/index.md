---
title: MS-TDS T-SQL 支持 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 MS-TDS T-SQL 支持。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 05/06/2019
ms.date: 09/24/2020
ms.openlocfilehash: cb012f7e09fd1b70d22cc8dc53c881639c518a4c
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146710"
---
# <a name="ms-tds-t-sql-support"></a>MS-TDS T-SQL 支持

Azure 数据资源管理器支持部分 Microsoft SQL Server 通信协议 (MS-TDS) 和部分 T-SQL 查询语言。 Microsoft Excel 和 Microsoft Power BI 只是可与 Azure 数据资源管理器配合使用的众多工具中的一部分。 这些 Microsoft 应用程序还知道如何查询 SQL Server。

> [!NOTE]
> 使用 Azure Active Directory (Azure AD) 集成的身份验证作为客户端工具可通过 MS-TDS 来查询 Kusto。

## <a name="next-steps"></a>后续步骤

* [T-SQL](./t-sql.md) - 了解由 Kusto 实现的 T-SQL 查询语言。 

* [基于 TDS 的 KQL](./tdskql.md) - 了解如何通过 TDS 终结点执行原生 KQL 查询。

* [MS-TDS 客户端和 Kusto](./clients.md) - 从使用 MS-TDS/T-SQL 的已知客户端使用 Azure 数据资源管理器。

* [使用 Azure 数据资源管理器作为链接服务器链接到 SQL Server](./linkedserver.md) - 将群集配置为链接服务器，以便链接到本地 SQL Server。 

* [将 MS-TDS 与 Azure Active Directory 配合使用](./aad.md) - 通过 TDS 使用 Azure AD，以便连接到 Azure 数据资源管理器。

* [SQL 已知问题](./sqlknownissues.md) - 了解 SQL Server 的 T-SQL 实现与 Azure 数据资源管理器之间的主要差异。
