---
title: Kusto 使用 AAD 进行身份验证以便进行访问 - Azure 数据资源管理器
description: 本文介绍了如何在 Azure 数据资源管理器中使用 AAD 进行身份验证以便访问 Azure 数据资源管理器。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref, devx-track-js
origin.date: 09/13/2019
ms.date: 10/29/2020
ms.openlocfilehash: bfc4b03410c12a39772753d92ea41e8e8f2dd28a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106315"
---
# <a name="how-to-authenticate-with-aad-for-azure-data-explorer-access"></a>如何使用 AAD 进行身份验证以便访问 Azure 数据资源管理器

访问 Azure 数据资源管理器的推荐方法是通过 **Azure Active Directory** 服务（有时也称为 **Azure AD** ，或简称为 **AAD** ）进行身份验证。 这样做可以通过一个两阶段过程来确保 Azure 数据资源管理器永远不会看到访问主体的目录凭据：

1. 在第一步中，客户端与 AAD 服务通信，向其证明身份，并请求一个针对客户端打算访问的特定 Azure 数据资源管理器终结点颁发的访问令牌。
2. 在第二步中，客户端向 Azure 数据资源管理器发出请求，并向 Azure 数据资源管理器提供在第一步中获取的访问令牌作为身份证明。

然后，Azure 数据资源管理器代表由 AAD 向其颁发访问令牌的安全主体执行请求，并使用此标识执行所有授权检查。

大多数情况下，建议使用 Azure 数据资源管理器 SDK 之一以编程方式访问该服务，因为它们避免了实现上述流（以及许多其他流）时的很多麻烦。 有关示例，请参阅 [.NET SDK](../../api/netfx/about-the-sdk.md)。
然后，通过 [Kusto 连接字符串](../../api/connection-strings/kusto.md)设置身份验证属性。
如果无法做到这一点，请继续阅读，详细了解如何自行实现此流程。

主要身份验证方案包括：

* **对登录用户进行身份验证的客户端应用程序** 。
  在此方案中，一个交互式（客户端）应用程序将触发 AAD 提示来提示用户输入凭据（例如用户名和密码）。
  请参阅[用户身份验证](#user-authentication)。

* **“无外设”应用程序** 。
  在此方案中，应用程序在运行时无需用户提供凭据，而是使用用户已为其配置的某些凭据，向 AAD 证明自己的身份。
  请参阅[应用程序身份验证](#application-authentication)。

* **代理身份验证** 。
  在此方案中（有时称为“Web 服务”或“Web 应用”方案），应用程序从另一个应用程序获取 AAD 访问令牌，然后将其“转换”为可与 Azure 数据资源管理器配合使用的另一个 AAD 访问令牌。
  换句话说，该应用程序充当提供了凭据的用户或应用程序与 Azure 数据资源管理器服务之间的中介。
  请参阅[代理身份验证](#on-behalf-of-authentication)。

## <a name="specifying-the-aad-resource-for-azure-data-explorer"></a>指定用于 Azure 数据资源管理器的 AAD 资源

从 AAD 获取访问令牌时，客户端必须告诉 AAD 应当将令牌颁发给哪个 **AAD 资源** 。 Azure 数据资源管理器终结点的 AAD 资源是终结点的 URI，其中不包括端口信息和路径。 例如：

```txt
https://help.kusto.chinacloudapi.cn
```



## <a name="specifying-the-aad-tenant-id"></a>指定 AAD 租户 ID

AAD 是一项多租户服务，每个组织都可以在 AAD 中创建一个称为 **目录** 的对象。 目录对象保存与安全性相关的对象，例如用户帐户、应用程序和组。 AAD 通常将目录称为 **租户** 。 AAD 租户通过 GUID（ **租户 ID** ）进行标识。 在许多情况下，AAD 租户还可以通过组织的域名进行标识。

例如，名为“Contoso”的组织可能具有租户 ID `4da81d62-e0a8-4899-adad-4349ca6bfe24` 和域名 `contoso.com`。

## <a name="specifying-the-aad-authority"></a>指定 AAD 颁发机构

AAD 有许多用于身份验证的终结点：

* 当承载着要进行身份验证的主体的租户已知时（换言之，当知道用户或应用程序在哪个 AAD 目录中时），AAD 终结点是 `https://login.partner.microsoftonline.cn/{tenantId}`。
  在这里，`{tenantId}` 是组织在 AAD 中的租户 ID，或者是组织的域名（例如 `contoso.com`）。

* 当承载着要进行身份验证的主体的租户未知时，可以通过将上面的 `{tenantId}` 替换为值 `common` 来使用“common”终结点。

> [!NOTE]
> 用于身份验证的 AAD 终结点也称为 **AAD 颁发机构 URL** 或简称为 **AAD 颁发机构** 。

## <a name="aad-token-cache"></a>AAD 令牌缓存

使用 Azure 数据资源管理器 SDK 时，AAD 令牌存储在本地计算机上每个用户的令牌缓存（一个名为 **%APPDATA%\Kusto\tokenCache.data** 的文件，该文件只能由已登录用户访问或解密）中。系统在提示用户输入凭据之前，会检查缓存中是否有令牌，从而大大减少用户必须输入凭据的次数。

> [!NOTE]
> AAD 令牌缓存可减少用户在访问 Azure 数据资源管理器时显示交互式提示的次数，但不会减少它们的填写。 此外，用户无法提前预测系统何时会提示其输入凭据。
> 这意味着，如果需要支持非交互式登录（例如，在计划任务时），则不能尝试使用用户帐户来访问 Azure 数据资源管理器，因为当提示登录用户输入凭据时，如果在非交互式登录下运行，则提示会失败。


## <a name="user-authentication"></a>用户身份验证

使用用户身份验证访问 Azure 数据资源管理器的最简单方法是使用 Azure 数据资源管理器 SDK，并将 Azure 数据资源管理器连接字符串的 `Federated Authentication` 属性设置为 `true`。 第一次使用 SDK 向服务发送请求时，系统会向用户显示一个用于输入 AAD 凭据的登录窗体，身份验证成功后请求就会发送。

不使用 Azure 数据资源管理器 SDK 的应用程序仍然可以使用 AAD 客户端库 (ADAL)，而不实现 AAD 服务安全协议客户端。 有关从 .NET 应用程序执行此操作的示例，请参阅 [https://github.com/AzureADSamples/WebApp-WebAPI-OpenIDConnect-DotNet ]。

若要针对 Azure 数据资源管理器访问对用户进行身份验证，必须先向应用程序授予 `Access Kusto` 委托权限。
以获取详细信息。

下面的简短代码片段演示了如何使用 ADAL 获取 AAD 用户令牌来访问 Azure 数据资源管理器（启动登录 UI）：

```csharp
// Create an HTTP request
WebRequest request = WebRequest.Create(new Uri("https://{serviceNameAndRegion}.kusto.chinacloudapi.cn"));

// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Acquire user token for the interactive user for Kusto:
AuthenticationResult result = authContext.AcquireTokenAsync("https://{serviceNameAndRegion}.kusto.chinacloudapi.cn", "your client app id",
    new Uri("your client app resource id"), new PlatformParameters(PromptBehavior.Auto)).GetAwaiter().GetResult();

// Extract Bearer access token and set the Authorization header on your request:
string bearerToken = result.AccessToken;
request.Headers.Set(HttpRequestHeader.Authorization, string.Format(CultureInfo.InvariantCulture, "{0} {1}", "Bearer", bearerToken));
```

## <a name="application-authentication"></a>应用程序身份验证

下面的简短代码片段演示了如何使用 ADAL 获取 AAD 应用程序令牌来访问 Azure 数据资源管理器。 此流中没有提示，应用程序必须向 AAD 注册并配备有执行应用程序身份验证所需的凭据（例如 AAD 颁发的应用密钥或已预先向 AAD 注册的 X509v2 证书）。

```csharp
// Create an HTTP request
WebRequest request = WebRequest.Create(new Uri("https://{serviceNameAndRegion}.kusto.chinacloudapi.cn"));

// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Acquire application token for Kusto:
ClientCredential applicationCredentials = new ClientCredential("your application client ID", "your application key");
AuthenticationResult result =
        authContext.AcquireTokenAsync("https://{serviceNameAndRegion}.kusto.chinacloudapi.cn", applicationCredentials).GetAwaiter().GetResult();

// Extract Bearer access token and set the Authorization header on your request:
string bearerToken = result.AccessToken;
request.Headers.Set(HttpRequestHeader.Authorization, string.Format(CultureInfo.InvariantCulture, "{0} {1}", "Bearer", bearerToken));
```

## <a name="on-behalf-of-authentication"></a>代理身份验证

在此方案中，向应用程序发送了由该应用程序管理的任意资源的 AAD 访问令牌，该应用程序使用该令牌为 Azure 数据资源管理器资源获取新的 AAD 访问令牌，以便应用程序可以代表原始 AAD 访问令牌指示的主体访问 Kusto。

此流称为 [OAuth2 令牌交换流](https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-04)。
它通常需要执行与 AAD 相关的多个配置步骤，并且在某些情况下（取决于 AAD 租户配置）可能需要 AAD 租户管理员的特别许可。



**步骤 1：在应用程序与 Azure 数据资源管理器服务之间建立信任关系**

1. 打开 [Azure 门户](https://portal.azure.cn/)，确保你已登录到正确的租户（在右上角查看用于登录门户的标识）。

2. 在资源窗格中，单击“Azure Active Directory”、“应用注册”。 

3. 找到使用代理流的应用程序并将其打开。

4. 单击“API 权限”，然后选择“添加权限”。

5. 搜索名为 **Azure 数据资源管理器** 的应用程序并选择它。

6. 选择 **user_impersonation / Access Kusto** 。

7. 单击“添加权限”。

**步骤 2：在服务器代码中执行令牌交换**

```csharp
// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Exchange your token for a Kusto token.
// You will need to provide your application's client ID and secret to authenticate your application
var tokenForKusto = authContext.AcquireTokenAsync(
    "https://{serviceNameAndRegion}.kusto.chinacloudapi.cn",
    new ClientCredential(customerAadWebApplicationClientId, customerAAdWebApplicationSecret),
    new UserAssertion(customerAadWebApplicationToken)).GetAwaiter().GetResult();
```

**步骤 3：向 Kusto 客户端库提供令牌并执行查询**

```csharp
var kcsb = new KustoConnectionStringBuilder(string.Format(
    "https://{0}.kusto.chinacloudapi.cn;fed=true;UserToken={1}",
    clusterName,
    tokenForKusto.AccessToken));
var client = KustoClientFactory.CreateCslQueryProvider(kcsb);
var queryResult = client.ExecuteQuery(databaseName, query, null);
```

## <a name="web-client-javascript-authentication-and-authorization"></a>Web 客户端 (JavaScript) 身份验证和授权

**详细信息**

如果客户端是在用户浏览器中运行的 JavaScript 代码，则使用隐式授权流。 在身份验证成功后，会立即在重定向 URI 中（在 URI 片段中）提供授权客户端应用程序访问 Azure 数据资源管理器服务的令牌；此流中不提供刷新令牌，因此客户端无法长时间缓存令牌并重新使用它。

与原生客户端流一样，应该有两个相互之间具有已配置的关系的 AAD 应用程序（服务器和客户端）。

AdalJs 需要在进行任何 access_token 调用之前获取 id_token。

访问令牌是通过调用 `AuthenticationContext.login()` 方法获取的，access_token 是通过调用 `Authenticationcontext.acquireToken()` 获取的。

* 使用正确的配置创建 AuthenticationContext：

```javascript
var config = {
    tenant: "microsoft.com",
    clientId: "<Web AAD app with current website as reply URL. for example, KusDash uses f86897ce-895e-44d3-91a6-aaeae54e278c>",
    redirectUri: "<where you'd like AAD to redirect after authentication succeeds (or fails).>",
    postLogoutRedirectUri: "where you'd like AAD to redirect the browser after logout."
};

var authContext = new AuthenticationContext(config);
```

* 如果尚未登录，请先调用 `authContext.login()`，然后再尝试 `acquireToken()`。 （若要了解你是否已登录，一个好办法是调用 `authContext.getCachedUser()`，看它是否返回 `false`）
* 每次加载页面时都调用 `authContext.handleWindowCallback()`。 这段代码从 AAD 截获重定向，然后从片段 URL 中提取令牌并缓存它。
* 调用 `authContext.acquireToken()` 以获取实际的访问令牌，此时你已拥有有效的 ID 令牌。 acquireToken 的第一个参数将是 Kusto 服务器 AAD 应用程序资源 URL。

```javascript
 authContext.acquireToken("<Kusto cluster URL>", callbackThatUsesTheToken);
 ```

* 在 callbackThatUsesTheToken 中，可以使用此令牌作为 Azure 数据资源管理器请求中的持有者令牌。 例如：

```javascript
var settings = {
    url: "https://" + clusterAndRegionName + ".kusto.chinacloudapi.cn/v1/rest/query",
    type: "POST",
    data: JSON.stringify({
        "db": dbName,
        "csl": query,
        "properties": null
    }),
    contentType: "application/json; charset=utf-8",
    headers: { "Authorization": "Bearer " + token },
    dataType: "json",
    jsonp: false,
    success: function(data, textStatus, jqXHR) {
        if (successCallback !== undefined) {
            successCallback(data.Tables[0]);
        }

    },
    error: function(jqXHR, textStatus, errorThrown) {
        if (failureCallback !==  undefined) {
            failureCallback(textStatus, errorThrown, jqXHR.responseText);
        }

    },
};

$.ajax(settings).then(function(data) {/* do something wil the data */});
```

> 警告 - 在进行身份验证时，如果收到以下异常或类似的异常：`ReferenceError: AuthenticationContext is not defined`，
可能是因为在全局命名空间中没有 AuthenticationContext。
遗憾的是，AdalJS 当前有一个未公开的要求，即身份验证上下文需在全局命名空间中定义。