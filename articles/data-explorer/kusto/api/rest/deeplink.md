---
title: UI 深层链接 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的 UI 深层链接。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 10/30/2019
ms.date: 10/29/2020
ms.openlocfilehash: af4de8194a7cb6ac9198b7d27babad5c8becd3eb
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103904"
---
# <a name="ui-deep-links"></a>UI 深层链接

REST API 提供了一个深层链接功能，该功能允许 HTTP `GET` 请求将调用方重定向到某个 UI 工具。 例如，你可以创建一个用于打开 Kusto.Explorer 工具的 URI，为特定群集和数据库自动配置该工具，运行特定查询并向用户显示其结果。

UI 深层链接 REST API：

* 群集（必需）通常隐式定义为实现了 REST API 的服务，但可通过指定 URI 查询参数 `uri` 来替代群集。

* 数据库（可选）指定为 URI 路径的第一个且唯一一个片段。 数据库对于查询来说是必需的，对于控制命令来说则是可选的。

* 查询或控制命令（可选）是使用 URI 查询参数 `query` 或 `querysrc`（指向保存查询的 Web 资源）指定的。
  `query` 可以在查询或控制命令本身的文本（使用 HTTP 查询参数编码方式进行编码）中使用。 此外，它还可以在查询或控制命令文本的 gzip 的 base64 编码中使用（可以压缩长查询，使其适合默认的浏览器 URI 长度限制）。

* 群集连接的名称（可选）是使用 URI 查询参数 `name` 指定的。

* UI 工具是使用 `web` 可选 URI 查询参数指定的。
  `web=0` 表示桌面应用程序 Kusto.Explorer。 `web=1` 表示 Kusto.WebExplorer Web 应用程序。
  `web=2` 是旧版 Kusto.WebExplorer（基于 Application Insights Analytics）。 `web=3` 是包含空配置文件的 Kusto.WebExplorer（之前打开的选项卡或群集都不可用）。 最后，可以将 `web` 查询参数替换为 `saw=1`，以指示 Kusto.Explorer 的 SAW 版本。

下面是一些链接示例：

* `https://help.kusto.chinacloudapi.cn/`：当用户代理（如浏览器）发出 `GET /` 请求时，该请求会被重定向到已配置的用来查询 `help` 群集的默认 UI 工具。
* `https://help.kusto.chinacloudapi.cn/Samples`：当用户代理（如浏览器）发出 `GET /Samples` 请求时，该请求会被重定向到已配置的用来查询 `help` 群集 `Samples` 数据库的默认 UI 工具。
* `http://help.kusto.chinacloudapi.cn/Samples?query=StormEvents`：当用户（如浏览器）发出 `GET /Samples?query=StormEvents` 请求时，该请求会被重定向到已配置的用来查询 `help` 群集 `Samples` 数据库的默认 UI 工具，并发出 `StormEvents` 查询。

> [!NOTE]
> 深层链接 URI 不需要进行身份验证，因为身份验证由用于重定向的 UI 工具执行。
> 任何 `Authorization` HTTP 标头（如果已提供）都会被忽略。

> [!IMPORTANT]
> 出于安全原因，UI 工具不会自动执行控制命令，即使在深层链接中指定了 `query` 或 `querysrc` 也是如此。

## <a name="deep-linking-to-kustoexplorer"></a>到 Kusto.Explorer 的深层链接

此 REST API 会执行重定向操作，该操作使用专门设计的启动参数（这些参数可打开与特定 Kusto 引擎群集的连接并对该群集执行查询）来安装和运行 Kusto.Explorer 桌面客户端工具。

* 路径：`/` [ *DatabaseName* `]
* 谓词：`GET`
* 查询字符串：`web=0`

> [!NOTE]
> 有关用于启动 Kusto.Explorer 的重定向 URI 语法的说明，请参阅[与 Kusto.Explorer 的深层链接](../../tools/kusto-explorer-using.md#deep-linking-queries)。

## <a name="deep-linking-to-kustowebexplorer"></a>到 Kusto.WebExplorer 的深层链接

此 REST API 执行到 Kusto.WebExplorer（一个 Web 应用程序）的重定向。

* 路径：`/` [ *DatabaseName* `]
* 谓词：`GET`
* 查询字符串：`web=1`

## <a name="specifying-the-query-or-control-command-in-the-uri"></a>在 URI 中指定查询或控制命令

当指定了 URI 查询字符串参数 `query` 时，必须根据 URI 查询字符串编码 HTML 规则对其进行编码。 也可通过 gzip 压缩查询或控制命令的文本，然后通过 base64 编码对其进行编码。 这样就可以发送更长的查询或控制命令（因为后一种编码方法会生成较短的 URI）。

## <a name="specifying-the-query-or-control-command-by-indirection"></a>通过间接寻址指定查询或控制命令

如果查询或控制命令非常长，即使使用 gzip/base64 对其进行编码也会超过用户代理的最大 URI 长度，那么， 也可提供 URI 查询字符串参数 `querysrc`，其值是一个短 URI，指向保存查询或控制命令文本的 Web 资源。

例如，它可以是 Azure Blob 存储所承载的文件的 URI。

> [!NOTE]
> 如果深层链接指向 Web 应用程序 UI 工具，则必须将提供查询或控制命令的 Web 服务（即提供 `querysrc` URI 的服务）配置为支持将 CORS 用于 `dataexplorer.azure.cn`。
>
> 此外，如果该服务需要身份验证/授权信息，则必须将其作为 URI 本身的一部分提供。
>
> 例如，如果 `querysrc` 指向 Azure Blob 存储中的 blob，则必须将存储帐户配置为支持 CORS，并使 blob 本身成为公共的（这样就可以在没有安全声明的情况下下载它），或将相应的 Azure 存储 SAS 添加到 URI。 可通过 [Azure 门户](https://portal.azure.cn/)或 [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)执行 CORS 配置。
> 请参阅 [Azure 存储中的 CORS 支持](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services)。
