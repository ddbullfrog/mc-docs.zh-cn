---
title: 将应用程序和 API 迁移到 b2clogin.cn
titleSuffix: Azure AD B2C
description: 了解如何在 Azure Active Directory B2C 的重定向 URL 中使用 b2clogin.cn。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/29/2020
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: 41c50777bc72d3a3625063b458d17b130dddfc07
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937100"
---
# <a name="set-redirect-urls-to-b2clogincn-for-azure-active-directory-b2c"></a>将 Azure Active Directory B2C 的重定向 URL 设置为 b2clogin.cn

在 Azure Active Directory B2C (Azure AD B2C) 应用程序中设置用于注册和登录的标识提供者时，需要指定一个重定向 URL。 请不要再在应用程序和 API 中引用 login.partner.microsoftonline.cn 来对用户进行 Azure AD B2C 身份验证。 对于所有新应用程序，应使用 *b2clogin.cn*，并将现有应用程序从 *login.partner.microsoftonline.cn* 迁移到 *b2clogin.cn*。

## <a name="benefits-of-b2clogincn"></a>b2clogin.cn 的优点

如果使用 *b2clogin.cn* 作为重定向 URL：

* Microsoft 服务在 cookie 标头中使用的空间就会减少。
* 重定向 URL 不再需要包含对 Microsoft 的引用。
* 自定义页面支持 JavaScript 客户端代码（目前为[预览版](user-flow-javascript-overview.md)）。 由于安全限制，如果使用 *login.partner.microsoftonline.cn*，将从自定义页面中删除 JavaScript 代码和 HTML 窗体元素。

## <a name="overview-of-required-changes"></a>所需的更改概述

若要将应用程序迁移到 *b2clogin.cn*，可能需要做出一些修改：

* 将标识提供者应用程序中的重定向 URL 更改为引用 *b2clogin.cn*。
* 将 Azure AD B2C 应用程序更新为在其用户流和令牌终结点引用中使用 *b2clogin.cn*。 这可能包括更新对诸如 Microsoft 身份验证库 (MSAL) 之类的身份验证库的使用。
* 更新在 `user interface customization` 的 CORS 设置中定义的任何“允许的来源”。

## <a name="change-identity-provider-redirect-urls"></a>更改标识提供者重定向 URL

在创建应用程序的每个标识提供者网站上，将所有受信任 URL 更改为重定向到 `your-tenant-name.b2clogin.cn` 而不是 *login.partner.microsoftonline.cn*。

对于 b2clogin.cn 重定向 URL，可以使用两种格式。 第一个选项使用租户 ID (GUID) 来替代租户域名，其优点是无需在 URL 中的任何位置显示“Microsoft”：

```
https://{your-tenant-name}.b2clogin.cn/{your-tenant-id}/oauth2/authresp
```

第二个选项使用 `your-tenant-name.partner.onmschina.cn` 格式的租户域名。 例如：

```
https://{your-tenant-name}.b2clogin.cn/{your-tenant-name}.partner.onmschina.cn/oauth2/authresp
```

对于这两种格式：

* 将 `{your-tenant-name}` 替换为 Azure AD B2C 租户的名称。
* 删除 `/te`（如果 URL 中存在此参数）。

## <a name="update-your-applications-and-apis"></a>更新应用程序和 API

已启用 Azure AD B2C 的应用程序和 API 中的代码可能在多个位置引用了 `login.partner.microsoftonline.cn`。 例如，代码可能引用了用户流和令牌终结点。 请更新以下代码，以改为引用 `your-tenant-name.b2clogin.cn`：

* 授权终结点
* 令牌终结点
* 令牌颁发者

例如，Contoso 注册/登录策略的机构终结点现在为：

```
https://contosob2c.b2clogin.cn/00000000-0000-0000-0000-000000000000/B2C_1_signupsignin1
```

如需了解如何迁移受 Azure AD B2C 保护的 Azure API 管理 API，请参阅[使用 Azure AD B2C 保护 Azure API 管理 API](secure-api-management.md) 的[迁移到 b2clogin.cn](secure-api-management.md#migrate-to-b2clogincom) 部分。

## <a name="microsoft-authentication-library-msal"></a>Microsoft 身份验证库 (MSAL)

### <a name="msalnet-validateauthority-property"></a>MSAL.NET ValidateAuthority 属性

如果使用 [MSAL.NET][msal-dotnet] v2 或更低版本，请在客户端实例化中将 **ValidateAuthority** 属性设置为 `false`，以允许重定向到 *b2clogin.cn*。 在 MSAL.NET v3 和更高版本中不需要将此值设置为 `false`。

```csharp
ConfidentialClientApplication client = new ConfidentialClientApplication(...); // Can also be PublicClientApplication
client.ValidateAuthority = false; // MSAL.NET v2 and earlier **ONLY**
```

### <a name="msal-for-javascript-validateauthority-property"></a>MSAL for JavaScript validateAuthority 属性

如果你使用的是 [MSAL for JavaScript][msal-js] v 1.2.2 或更早版本，请将“validateAuthority”属性设置为 `false`。

```JavaScript
// MSAL.js v1.2.2 and earlier
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: false // Required in MSAL.js v1.2.2 and earlier **ONLY**
  }
);
```

如果在 MSAL.js 1.3.0+（默认值）中设置 `validateAuthority: true`，则还需要使用 `knownAuthorities` 指定有效的令牌颁发者：

```JavaScript
// MSAL.js v1.3.0+
this.clientApplication = new UserAgentApplication(
  env.auth.clientId,
  env.auth.loginAuthority,
  this.authCallback.bind(this),
  {
    validateAuthority: true, // Supported in MSAL.js v1.3.0+
    knownAuthorities: ['tenant-name.b2clogin.cn'] // Required if validateAuthority: true
  }
);
```

## <a name="next-steps"></a>后续步骤

如需了解如何迁移受 Azure AD B2C 保护的 Azure API 管理 API，请参阅[使用 Azure AD B2C 保护 Azure API 管理 API](secure-api-management.md) 的[迁移到 b2clogin.cn](secure-api-management.md#migrate-to-b2clogincom) 部分。

<!-- LINKS - External -->
[msal-dotnet]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[msal-dotnet-b2c]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics
[msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
[msal-js-b2c]: ../active-directory/develop/msal-b2c-overview.md

