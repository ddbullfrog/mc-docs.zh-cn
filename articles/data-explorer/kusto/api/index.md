---
title: Azure 数据资源管理器 API 概述 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 API。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: vladikb
ms.service: data-explorer
ms.topic: reference
origin.date: 08/11/2020
ms.date: 09/24/2020
ms.openlocfilehash: ca6121f16ad752a64fd17607b98f17430b0321df
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146787"
---
# <a name="azure-data-explorer-api-overview"></a>Azure 数据资源管理器 API 概述

Azure 数据资源管理器服务支持以下通信终结点：

1. 一个 [REST API](#rest-api) 终结点，你可以通过它查询和管理 Azure 数据资源管理器中的数据。
   此终结点支持用于查询的 [Kusto 查询语言](../query/index.md)以及[控制命令](../management/index.md)。
1. 一个 [MS-TDS](#ms-tds) 终结点，用于实现部分 Microsoft 表格格式数据流 (TDS) 协议，供 Microsoft SQL Server 产品使用。
   此终结点对知道如何与 SQL Server 终结点通信来进行查询的工具很有用。
1. 一个 [Azure 资源管理器 (ARM)](https://docs.azure.cn/role-based-access-control/resource-provider-operations#microsoftkusto) 终结点，它是 Azure 服务的标准方式。 该终结点用于管理 Azure 数据资源管理器群集等资源。

## <a name="rest-api"></a>REST API

与任何 Azure 数据资源管理器服务通信时，主要方式是使用服务的 REST API。 通过这个完全记录的终结点，调用方可：

* 查询数据
* 查询和修改元数据
* 引入数据
* 查询服务运行状况
* 管理资源

不同的 Azure 数据资源管理器服务使用同一个公开提供的 REST API 相互通信。

还可通过许多[客户端库](client-libraries.md)来使用服务，不需处理 REST API 协议。

## <a name="ms-tds"></a>MS-TDS

Azure 数据资源管理器还支持 Microsoft SQL Server 通信协议 (MS-TDS)，同时为运行 T-SQL 查询提供有限支持。 通过此协议，用户可使用众所周知的查询语法 (T-SQL) 和数据库客户端工具（例如 LINQPad、sqlcmd、Tableau、Excel 和 Power BI）在 Azure 数据资源管理器上运行查询。

有关详细信息，请参阅 [MS-TDS](tds/index.md)。

## <a name="client-libraries"></a>客户端库 

Azure 数据资源管理器提供了很多使用上述终结点的客户端库，让编程性访问变得容易起来。

* .NET SDK
* Python SDK
* R
* Java SDK
* Node SDK
* Go SDK
* PowerShell

### <a name="net-framework-libraries"></a>.NET Framework 库

若要以编程方式调用 Azure 数据资源管理器功能，建议使用 .NET Framework 库。
有很多不同的库可供使用。

* [Kusto.Data（Kusto 客户端库）](./netfx/about-kusto-data.md)：可用于查询数据、查询元数据并对其进行更改。 
   它构建在 Kusto REST API 基础之上，可将 HTTPS 请求发送到目标 Kusto 群集。
* [Kusto.Ingest（Kusto 引入库）](netfx/about-kusto-ingest.md)：使用 `Kusto.Data` 并扩展它来简化数据引入。

上述库使用 Azure API（例如 Azure 存储 API 和 Azure Active Directory API）。

### <a name="python-libraries"></a>Python 库

Azure 数据资源管理器提供 Python 客户端库，让调用方能够发送数据查询和控制命令。
有关详细信息，请参阅 [Azure 数据资源管理器 Python SDK](python/kusto-python-client-library.md)。

### <a name="r-library"></a>R 库

Azure 数据资源管理器提供 R 客户端库，让调用方能够发送数据查询和控制命令。
有关详细信息，请参阅 [Azure 数据资源管理器 R SDK](r/kusto-r-client-library.md)。

### <a name="java-sdk"></a>Java SDK

通过 Java 客户端库，可使用 Java 查询 Azure 数据资源管理器群集。 有关详细信息，请参阅 [Azure 数据资源管理器 Java SDK](java/kusto-java-client-library.md)。

### <a name="node-sdk"></a>Node SDK

Azure 数据资源管理器 Node SDK 与 Node LTS（当前为 v6.14）兼容，通过 ES6 生成。
有关详细信息，请参阅 [Azure 数据资源管理器 Node SDK](node/kusto-node-client-library.md)。

### <a name="go-sdk"></a>Go SDK

Azure 数据资源管理器 Go 客户端库提供了使用 Go 查询、控制 Azure 数据资源管理器群集以及将数据引入其中的功能。 有关详细信息，请参阅 [Azure 数据资源管理器 Golang SDK](golang/kusto-golang-client-library.md)。

### <a name="powershell"></a>PowerShell

Azure 数据资源管理器 .NET Framework 库可供 PowerShell 脚本使用。 有关详细信息，请参阅[从 PowerShell 调用 Azure 数据资源管理器](powershell/powershell.md)。

## <a name="monaco-ide-integration"></a>Monaco IDE 集成

`monaco-kusto` 包支持与 Monaco Web 编辑器的集成。
Monaco 编辑器由 Microsoft 开发，是 Visual Studio Code 的基础。
有关详细信息，请参阅 [monaco-kusto 包](monaco/monaco-kusto.md)。
