---
title: 在 API 管理中使用 OAuth 2.0 为开发人员帐户授权
titleSuffix: Azure API Management
description: 了解如何在 API 管理中使用 OAuth 2.0 为用户授权。 OAuth 2.0 保护 API，以便用户只能访问他们有权访问的资源。
services: api-management
documentationcenter: ''
author: Johnnytechn
manager: cfowler
editor: ''
origin.date: 11/04/2019
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 09/29/2020
ms.author: v-johya
ms.openlocfilehash: 4c80a3fd3c08000268d586caf7e97af67cb4263c
ms.sourcegitcommit: 80567f1c67f6bdbd8a20adeebf6e2569d7741923
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871374"
---
# <a name="how-to-authorize-developer-accounts-using-oauth-20-in-azure-api-management"></a>如何在 Azure API 管理中使用 OAuth 2.0 为开发人员帐户授权

许多 API 支持使用 [OAuth 2.0](https://oauth.net/2/) 维护 API 的安全，并确保仅有效用户具有访问权限且只能访问有权访问的资源。 要将 Azure API 管理的交互式开发人员门户与此类 API 配合使用，需通过该服务对服务实例进行配置，使之适用于支持 OAuth 2.0 的 API。

> [!IMPORTANT]
> OAuth 2.0 授权在新开发人员门户的交互式控制台中尚不可用。

## <a name="prerequisites"></a><a name="prerequisites"> </a>先决条件

本指南介绍如何配置 API 管理服务实例，以便针对开发人员帐户使用 OAuth 2.0 授权，但不介绍如何配置 OAuth 2.0 提供程序。 每个 OAuth 2.0 提供程序的配置均不相同，虽然步骤类似，不过在 API 管理服务实例中配置 OAuth 2.0 时使用的必需信息是相同的。 本主题介绍的示例使用 Azure Active Directory 作为 OAuth 2.0 提供程序。

> [!NOTE]
> 有关使用 Azure Active Directory 配置 OAuth 2.0 的详细信息，请参阅 [WebApp-GraphAPI-DotNet][WebApp-GraphAPI-DotNet] 示例。

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="configure-an-oauth-20-authorization-server-in-api-management"></a><a name="step1"> </a>在 API 管理中配置 OAuth 2.0 授权服务器

> [!NOTE]
> 如果尚未创建 API 管理服务实例，请参阅[创建 API 管理服务实例][Create an API Management service instance]。

1. 单击左侧菜单中的 OAuth 2.0 标签，然后单击“+添加”****。

    ![OAuth 2.0 菜单](./media/api-management-howto-oauth2/oauth-01.png)

2. 在“名称”**** 和“说明”**** 字段中输入名称和可选说明。

    > [!NOTE]
    > 这些字段用于标识当前 API 管理服务实例中的 OAuth 2.0 授权服务器，其值不来自 OAuth 2.0 服务器。

3. 输入“客户端注册页 URL”。**** 此页是供用户创建和管理其帐户的地方，因所使用的 OAuth 2.0 提供程序而异。 “客户端注册页 URL”指向供用户针对 OAuth 2.0 提供程序创建和配置自己帐户的页面，这些提供程序支持用户管理帐户，例如 `https://contoso.com/login`。 某些组织不配置或使用此功能，即使 OAuth 2.0 提供程序支持此功能。 如果 OAuth 2.0 提供程序尚未配置用户管理帐户功能，请在此处输入一个占位符 URL，例如公司的 URL，或 `https://placeholder.contoso.com` 之类的 URL。

    ![OAuth 2.0 新服务器](./media/api-management-howto-oauth2/oauth-02.png)

4. 此窗体的下一部分包含“授权的授权类型”、“授权终结点 URL”和“授权请求方法”设置。************

    选中所需类型即可指定“授权的授权类型”。**** “授权代码”是默认指定的。****

    输入“授权终结点 URL”。**** 对于 Azure Active Directory，此 URL 将类似于以下 URL，其中 `<tenant_id>` 将替换为 Azure AD 租户的 ID。

    `https://login.partner.microsoftonline.cn/<tenant_id>/oauth2/authorize`

    “授权请求方法”指定如何向 OAuth 2.0 服务器发送授权请求。**** 默认选择 **GET**。

5. 然后，需要指定“令牌终结点 URL”、“客户端身份验证方法”、“访问令牌发送方法”和“默认范围”。****************

    ![OAuth 2.0 新服务器](./media/api-management-howto-oauth2/oauth-03.png)

    对于 Azure Active Directory OAuth 2.0 服务器，“令牌终结点 URL”将具有如下格式，其中 `<TenantID>` 的格式为 `yourapp.partner.onmschina.cn`。****

    `https://login.partner.microsoftonline.cn/<TenantID>/oauth2/token`

    “客户端身份验证方法”的默认设置为“基本”，“访问令牌发送方法”为“Authorization 标头”。**************** 这些值以及“默认范围”在窗体的此部分配置。****

6. “客户端凭据”部分包含“客户端 ID”和“客户端密钥”，在创建和配置 OAuth 2.0 服务器的过程中获取。************ 指定“客户端 ID”和“客户端密钥”以后，会生成“授权代码”的“redirect_uri”。**************** 该 URI 用于在 OAuth 2.0 服务器配置中配置回复 URL。

    在新的开发人员门户中，URI 后缀的形式为：

    - `/signin-oauth/code/callback/{authServerName}` - 授权代码授予流
    - `/signin-oauth/implicit/callback` - 隐式授权流

    ![OAuth 2.0 新服务器](./media/api-management-howto-oauth2/oauth-04.png)

    如果“授权的授权类型”设置为“资源所有者密码”，则可使用“资源所有者密码凭据”部分指定这些凭据；否则可将其留空。************

    完成窗体的操作后，单击“创建”保存 API 管理 OAuth 2.0 授权服务器配置。**** 保存服务器配置后，可将 API 配置为使用此配置，如下一部分所示。

## <a name="configure-an-api-to-use-oauth-20-user-authorization"></a><a name="step2"> </a>将 API 配置为使用 OAuth 2.0 用户授权

1. 单击左侧“API 管理”**** 菜单中的“API”****。

    ![OAuth 2.0 API](./media/api-management-howto-oauth2/oauth-05.png)

2. 单击所需 API 的名称，然后单击“设置”****。 滚动至“安全性”**** 部分，然后选中 **OAuth 2.0** 框。

    ![OAuth 2.0 设置](./media/api-management-howto-oauth2/oauth-06.png)

3. 从下拉列表中选择所需的“授权服务器”，并单击“保存”。********

    ![OAuth 2.0 设置](./media/api-management-howto-oauth2/oauth-07.png)

## <a name="legacy-developer-portal---test-the-oauth-20-user-authorization"></a><a name="step3"> </a>旧开发人员门户 - 测试 OAuth 2.0 用户授权

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

配置 OAuth 2.0 授权服务器并将 API 配置为使用该服务器以后，即可转到开发人员门户并调用 API 对其进行测试。 在 Azure API 管理实例“概述”页的顶部菜单中，单击“开发人员门户(旧)”。********

单击顶部菜单中的“API”，并选择“Echo API”。********

![Echo API][api-management-apis-echo-api]

> [!NOTE]
> 如果必须只有一个 API 得到配置或对你的帐户可见，并单击 API 使你直接进入该 API 的操作。

选择“GET 资源”操作，单击“打开控制台”，并从下拉列表中选择“授权代码”。************

![打开控制台][api-management-open-console]

选中“授权代码”后，会显示一个弹出窗口，其中包含 OAuth 2.0 提供程序的登录窗体。**** 在此示例中，登录窗体由 Azure Active Directory 提供。

> [!NOTE]
> 如果已禁用弹出窗口，浏览器会提示用户启用该功能。 启用该功能后，再次选中“授权代码”，此时就会显示登录窗体。****

![登录][api-management-oauth2-signin]

登录后，“请求标头”中会填充用于对请求授权的 `Authorization : Bearer` 标头。****

![请求标头令牌][api-management-request-header-token]

此时可以配置剩余参数的所需值，并提交请求。

## <a name="next-steps"></a>后续步骤

有关如何使用 OAuth 2.0 和 API 管理的详细信息，请观看以下视频并查看随附的[文章](api-management-howto-protect-backend-with-aad.md)。

[api-management-oauth2-signin]: ./media/api-management-howto-oauth2/api-management-oauth2-signin.png
[api-management-request-header-token]: ./media/api-management-howto-oauth2/api-management-request-header-token.png
[api-management-open-console]: ./media/api-management-howto-oauth2/api-management-open-console.png
[api-management-apis-echo-api]: ./media/api-management-howto-oauth2/api-management-apis-echo-api.png

[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[https://oauth.net/2/]: https://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

