---
title: sql_request 插件 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 sql_request 插件。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/24/2020
ms.date: 09/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 213fb99b7c26a88e5ca8662a72ab2541c8e140e4
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103570"
---
# <a name="sql_request-plugin"></a>sql_request 插件

::: zone pivot="azuredataexplorer"

`sql_request` 插件将 SQL 查询发送到 SQL Server 网络终结点，并返回结果中的第一个行集。

## <a name="syntax"></a>语法

  `evaluate` `sql_request` `(` *ConnectionString* `,` *SqlQuery* [`,` *SqlParameters* [`,` *Options* ]] `)`

## <a name="arguments"></a>参数

* ConnectionString：一个 `string` 文本，用于指示指向 SQL Server 网络终结点的连接字符串。 请了解[有效的身份验证方法](#authentication)以及指定[网络终结点](#specify-the-network-endpoint)的方法。

* SqlQuery：一个 `string` 文本，用于指示要对 SQL 终结点执行的查询。 必须返回一个或多个行集，但只有第一个行集可用于 Kusto 查询的其余部分。

* SqlParameters：`dynamic` 类型的常数值，用于保存作为参数随查询传递的键值对。 可选。
  
* *选项* ：`dynamic` 类型的常数值，它将更高级的设置保存为键值对。 目前只能设置 `token`，以便传递调用方提供的 Azure AD 访问令牌，该令牌将转发到 SQL 终结点，用于身份验证。 可选。

## <a name="examples"></a>示例

下面的示例向 Azure SQL DB 数据库发送 SQL 查询。 它从 `[dbo].[Table]` 检索所有记录，然后在 Kusto 端处理结果。 身份验证会重复使用调用方用户的 Azure AD 令牌。 

> [!NOTE]
> 建议不要像此示例这样以此方式筛选或投影数据。 在构造 SQL 查询时，应当尽可能返回最小的数据集，因为 Kusto 优化器当前不会尝试优化 Kusto 与 SQL 之间的查询。

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.chinacloudapi.cn,1433;'
    'Authentication="Active Directory Integrated";'
    'Initial Catalog=Fabrikam;',
  'select * from [dbo].[Table]')
| where Id > 0
| project Name
```

下面的示例与上一个示例相同，只不过 SQL 身份验证是通过用户名/密码来执行的。 考虑到保密性，我们在此处使用经过模糊处理的字符串。

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.chinacloudapi.cn,1433;'
    'Initial Catalog=Fabrikam;'
    h'User ID=USERNAME;'
    h'Password=PASSWORD;',
  'select * from [dbo].[Table]')
| where Id > 0
| project Name
```

下面的示例向 Azure SQL DB 数据库发送一个 SQL 查询，从 `[dbo].[Table]` 检索所有记录，同时追加另一个 `datetime` 列，然后在 Kusto 端处理结果。
它指定要在 SQL 查询中使用的 SQL 参数 (`@param0`)。

```kusto
evaluate sql_request(
  'Server=tcp:contoso.database.chinacloudapi.cn,1433;'
    'Authentication="Active Directory Integrated";'
    'Initial Catalog=Fabrikam;',
  'select *, @param0 as dt from [dbo].[Table]',
  dynamic({'param0': datetime(2020-01-01 16:47:26.7423305)}))
| where Id > 0
| project Name
```

## <a name="authentication"></a>身份验证

sql_request 插件支持对 SQL Server 终结点使用以下三种身份验证方法：

### <a name="azure-ad-integrated-authentication"></a>Azure AD 集成式身份验证 

`Authentication="Active Directory Integrated"`

  Azure AD 集成式身份验证是首选方法。 此方法让用户或应用程序通过 Azure AD 向 Kusto 进行身份验证。 然后，使用同一令牌访问 SQL Server 网络终结点。

### <a name="usernamepassword-authentication"></a>用户名/密码身份验证

`User ID=...; Password=...;`

  当无法执行 Azure AD 集成式身份验证时，会提供用户名和密码身份验证支持。 如果可能，请尽量避免使用此方法，因为机密信息是通过 Kusto 发送的。

### <a name="azure-ad-access-token"></a>Azure AD 访问令牌

`dynamic({'token': h"eyJ0..."})`

   使用 Azure AD 访问令牌身份验证方法时，调用方会生成访问令牌，该令牌由 Kusto 转发到 SQL 终结点。 不应在连接字符串中包含身份验证信息（例如 `Authentication`、`User ID` 或 `Password`）， 而应将访问令牌作为 sql_request 插件的 `Options` 参数中的 `token` 属性传递。
     
> [!WARNING]
> 应该对包含机密信息（或应当受保护的信息）的连接字符串和查询进行模糊处理，以在任何 Kusto 跟踪中忽略它们。
> 有关详细信息，请参阅[经过模糊处理的字符串文本](scalar-data-types/string.md#obfuscated-string-literals)。

## <a name="encryption-and-server-validation"></a>加密和服务器验证

出于安全方面的原因，连接到 SQL Server 网络终结点时，会强制实施以下连接属性。

* `Encrypt` 无条件地设置为 `true`。
* `TrustServerCertificate` 无条件地设置为 `false`。

因此，必须为 SQL Server 配置有效的 SSL/TLS 服务器证书。

## <a name="specify-the-network-endpoint"></a>指定网络终结点

必须将 SQL 网络终结点指定为连接字符串的一部分。
正确的语法为：

`Server` `=` `tcp:` *FQDN* [`,` *Port* ]

其中：

* *FQDN* 是终结点的完全限定域名。
* *Port* 是终结点的 TCP 端口。 默认为 `1433`。

> [!NOTE]
> 不支持以其他形式指定网络终结点。
> 例如，不能省略前缀 `tcp:`，即使在以编程方式使用 SQL 客户端库时可以省略它。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
