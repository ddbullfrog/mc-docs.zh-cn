---
title: 将调用 Web API 的桌面应用移到生产环境 - Microsoft 标识平台 | Azure
description: 了解如何将调用 Web API 的桌面应用移到生产环境
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/06/2020
ms.author: v-junlch
ms.custom: aaddev
ms.openlocfilehash: aa2f0104d5af3713f8e2da4a147ed79283676e8e
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211490"
---
# <a name="desktop-app-that-calls-web-apis-move-to-production"></a>用于调用 Web API 的桌面应用：移到生产环境

本文介绍如何将调用 Web API 的桌面应用移到生产环境。

## <a name="handle-errors-in-desktop-applications"></a>在桌面应用程序中处理错误

你已了解，在不同的流中如何处理静默流的错误，如代码片段所示。 你还了解，有些情况下需要交互（与增量许可中的情况一样）。

## <a name="have-the-user-consent-upfront-for-several-resources"></a>让用户提前许可多个资源

> [!NOTE]
> 获得多个资源的同意适用于 Microsoft 标识平台，但不适用于 Azure Active Directory (Azure AD) B2C。 Azure AD B2C 仅支持管理员同意，不支持用户同意。

不能使用 Microsoft 标识平台 (v2.0) 终结点一次获取多个资源的令牌。 `scopes` 参数只能包含单个资源的范围。 可以使用 `extraScopesToConsent` 参数确保用户预先同意多个资源。

例如，你可能有两个资源（每个资源有两个范围）：

- `https://mytenant.partner.onmschina.cn/customerapi`，范围为 `customer.read` 和 `customer.write`
- `https://mytenant.partner.onmschina.cn/vendorapi`，范围为 `vendor.read` 和 `vendor.write`

在此示例中，请使用具有 `extraScopesToConsent` 参数的 `.WithAdditionalPromptToConsent` 修饰符。

例如：

### <a name="in-msalnet"></a>在 MSAL.NET 中

```csharp
string[] scopesForCustomerApi = new string[]
{
  "https://mytenant.partner.onmschina.cn/customerapi/customer.read",
  "https://mytenant.partner.onmschina.cn/customerapi/customer.write"
};
string[] scopesForVendorApi = new string[]
{
 "https://mytenant.partner.onmschina.cn/vendorapi/vendor.read",
 "https://mytenant.partner.onmschina.cn/vendorapi/vendor.write"
};

var accounts = await app.GetAccountsAsync();
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithAccount(accounts.FirstOrDefault())
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

### <a name="in-msal-for-ios-and-macos"></a>在适用于 iOS 和 macOS 的 MSAL 中

Objective-C：

```objc
NSArray *scopesForCustomerApi = @[@"https://mytenant.partner.onmschina.cn/customerapi/customer.read",
                                @"https://mytenant.partner.onmschina.cn/customerapi/customer.write"];

NSArray *scopesForVendorApi = @[@"https://mytenant.partner.onmschina.cn/vendorapi/vendor.read",
                              @"https://mytenant.partner.onmschina.cn/vendorapi/vendor.write"]

MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopesForCustomerApi webviewParameters:[MSALWebviewParameters new]];
interactiveParams.extraScopesToConsent = scopesForVendorApi;
[application acquireTokenWithParameters:interactiveParams completionBlock:^(MSALResult *result, NSError *error) { /* handle result */ }];
```

Swift：

```swift
let scopesForCustomerApi = ["https://mytenant.partner.onmschina.cn/customerapi/customer.read",
                            "https://mytenant.partner.onmschina.cn/customerapi/customer.write"]

let scopesForVendorApi = ["https://mytenant.partner.onmschina.cn/vendorapi/vendor.read",
                          "https://mytenant.partner.onmschina.cn/vendorapi/vendor.write"]

let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopesForCustomerApi, webviewParameters: MSALWebviewParameters())
interactiveParameters.extraScopesToConsent = scopesForVendorApi
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in /* handle result */ })
```

此调用为你获得第一个 Web API 的访问令牌。

如果需要调用第二个 Web API，请调用 `AcquireTokenSilent` API。

```csharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Move to production common steps](../../../includes/active-directory-develop-scenarios-production.md)]

<!-- Update_Description: wording update -->