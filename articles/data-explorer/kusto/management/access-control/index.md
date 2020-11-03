---
title: Kusto 访问控制概述 - Azure 数据资源管理器
description: 本文概述了 Azure 数据资源管理器中的 Kusto 访问控制。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
origin.date: 11/25/2019
ms.date: 10/29/2020
ms.openlocfilehash: 415bd8da56d9aefa64899b5867a8fff1c4cf1e37
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106314"
---
# <a name="kusto-access-control-overview"></a>Kusto 访问控制概述

Azure 数据资源管理器中的访问控制基于两个关键因素。
* [身份验证](#authentication)：验证进行请求的安全主体的标识
* [授权](#authorization)：验证进行请求的安全主体是否获允在目标资源上进行该请求

Azure 数据资源管理器群集、数据库或表上的查询或控制命令必须同时通过身份验证和授权检查。

## <a name="authentication"></a>身份验证

Azure Active Directory (Azure AD) 是 Azure 的首选多租户云目录服务。 它可对安全主体进行身份验证或与其他标识提供者联合。

Azure AD 是在 Microsoft 中向 Azure 数据资源管理器进行身份验证的首选方法。 它支持多种身份验证方案。
* 用户身份验证（交互式登录）：用于对人类主体进行身份验证。
* 应用程序身份验证（非交互式登录）：用于对必须在没有人类用户参与的情况下运行或进行身份验证的服务和应用程序进行身份验证。

### <a name="user-authentication"></a>用户身份验证

用户身份验证是在用户向以下应用提供凭据时完成的：
* Azure AD 
* 与 Azure AD 一起使用的标识提供者

如果成功，用户将收到一个安全令牌，它可提供给 Azure 数据资源管理器服务。 Azure 数据资源管理器服务并不关心安全令牌是如何获取的。 它关心的是令牌是否有效，以及 Azure AD（或进行联合身份验证的 IdP）在令牌中放入了哪些信息。

在客户端，Azure 数据资源管理器支持交互式身份验证，而 Microsoft 身份验证库或类似的代码会要求用户输入凭据。 它还支持基于令牌的身份验证，其中使用 Azure 数据资源管理器的应用程序会获取有效的用户令牌。 使用 Azure 数据资源管理器的应用程序也可为其他服务获取有效的用户令牌。 只有当该资源与 Azure 数据资源管理器之间存在信任关系时，才可获取用户令牌。

有关详细信息，请参阅 [Kusto 连接字符串](../../api/connection-strings/kusto.md)，详细了解如何使用 Kusto 客户端库并使用 Azure AD 向 Azure 数据资源管理器进行身份验证。

### <a name="application-authentication"></a>应用程序身份验证

如果请求与特定用户不相关，或者没有用户来输入凭据，则请使用 Azure AD 应用程序身份验证流。 在该流中，应用程序通过提供某种机密信息向 Azure AD（或进行联合身份验证的 IdP）进行身份验证。 可使用各种 Azure 数据资源管理器客户端实现以下方案。

* 使用安装在本地的 X.509v2 证书进行的应用程序身份验证
* 使用作为字节流提供给客户端库的 X.509v2 证书进行的应用程序身份验证
* 使用 Azure AD 身份验证 ID 和 Azure AD 身份验证密钥进行的应用程序身份验证。

    > [!NOTE] 
    > 这里的 ID 和密钥等效于用户名和密码

* 使用之前获得的有效的 Azure AD 令牌（颁发给 Azure 数据资源管理器）进行的应用程序身份验证。
* 使用之前获得的有效的 Azure AD 令牌（颁发给其他某项资源）进行的应用程序身份验证。 如果该资源与 Azure 数据资源管理器之间存在信任关系，则可使用此方法。

### <a name="microsoft-accounts-msas"></a>Microsoft 帐户 (MSA)

Microsoft 帐户 (MSA) 这个术语是指所有由 Microsoft 托管的非组织用户帐户，例如 `hotmail.com`、`live.com`、`outlook.com`。
Kusto 支持对 MSA 进行用户身份验证（这里没有“安全组”这一概念），其中 MSA 按用户主体名称 (UPN) 进行标识。

在 Azure 数据资源管理器资源上配置 MSA 主体时，该资源管理器不会尝试解析所提供的 UPN。

### <a name="authenticated-sdk-or-rest-calls"></a>经过身份验证的 SDK 或 REST 调用

* 使用 REST API 时，身份验证是通过标准 HTTP `Authorization` 标头进行的
* 使用任何 Azure 数据资源管理器 .NET 库时，可通过在[连接字符串](../../api/connection-strings/kusto.md)中指定身份验证方法和参数来控制身份验证。 另一种方法是在[客户端请求属性](../../api/netfx/request-properties.md)对象上设置属性。

### <a name="azure-data-explorer-client-sdk-as-an-azure-ad-client-application"></a>Azure 数据资源管理器客户端 SDK 用作 Azure AD 客户端应用程序

Kusto 客户端库在调用 Microsoft 身份验证库来获取与 Kusto 通信所需的令牌时，会提供以下信息：

* 资源（群集 URI，例如 `https://Cluster-and-region.kusto.chinacloudapi.cn`）
* Azure AD 客户端应用程序 ID
* Azure AD 客户端应用程序重定向 URI
* Azure AD 租户，它会影响用于身份验证的 Azure AD 终结点。 例如，对于 Azure AD 租户 `microsoft.com`，Azure AD 终结点为 `https://login.partner.microsoftonline.cn/microsoft.com`

Microsoft 身份验证库向 Azure 数据资源管理器客户端库返回的令牌具有适当的 Azure 数据资源管理器群集 URL 作为受众，且具有“访问 Azure 数据资源管理器”权限作为范围。

**示例：为 Azure 数据资源管理器群集获取 Azure AD 用户令牌**

```csharp
// Create Auth Context for Azure AD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("https://login.partner.microsoftonline.cn/{Azure AD TenantID or name}");

// Provide your Application ID and redirect URI
var clientAppID = "{your client app id}";
var redirectUri = new Uri("{your client app redirect uri}");

// acquireTokenTask will receive the bearer token for the authenticated user
var acquireTokenTask = authContext.AcquireTokenAsync(
    $"https://{clusterNameAndRegion}.kusto.chinacloudapi.cn",
    clientAppID,
    redirectUri,
    new PlatformParameters(PromptBehavior.Auto, null)).GetAwaiter().GetResult();
```

## <a name="authorization"></a>授权

所有经过身份验证的主体都要先接受授权检查，然后才能对 Azure 数据资源管理器资源执行操作。
Azure 数据资源管理器使用一种[基于角色的授权模型](role-based-authorization.md)，其中主体归属一个或多个安全角色。 只要主体的一个角色获得授权，授权就会成功。

例如，数据库用户角色会向安全主体、用户或服务授予以下权限：
* 读取特定数据库的数据
* 在数据库中创建表
* 在数据库中创建函数

将安全主体关联到安全角色时，可单独进行定义，也可使用安全组进行定义（在 Azure AD 中定义）。 这些命令是在[设置基于角色的授权规则](../security-roles.md)中定义的。
