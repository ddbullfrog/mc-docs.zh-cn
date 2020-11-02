---
title: Kusto Azure Active Directory (AAD) 身份验证 - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 Azure Active Directory (AAD) 身份验证。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
origin.date: 09/13/2019
ms.date: 10/29/2020
ms.openlocfilehash: ed67f91120ffe4e4bf003289b0ba754d98eb25a8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105530"
---
# <a name="azure-active-directory-aad-authentication"></a>Azure Active Directory (AAD) 身份验证

Azure Active Directory (AAD) 是 Azure 的首选多租户云目录服务，可以对安全主体单独进行身份验证，还可以与其他标识提供者（例如 Microsoft 的 Active Directory）联合进行身份验证。

各种应用程序（Web 应用程序、Windows 桌面应用程序、通用应用程序、移动应用程序等）可以通过 AAD 进行统一身份验证和使用 Kusto 服务。

AAD 支持多种身份验证方案。
如果在身份验证过程中存在用户，则应通过 AAD 用户身份验证向 AAD 验证该用户的身份。
在某些情况下，即使没有用户交互，也需要通过服务来使用 Kusto。 在这种情况下，应通过使用应用程序机密来验证应用程序，如“AAD 应用程序身份验证”中所述。

通常，Kusto 支持以下身份验证方法（包括通过其 .NET 库进行的身份验证方法）：

* 交互式用户身份验证 - 此模式需要交互，在需要的情况下登录 UI 会弹出
* 使用之前为 Kusto 颁发的现有 AAD 令牌进行用户身份验证
* 使用 AppID 和共享机密进行应用程序身份验证
* 使用本地安装的 X.509v2 证书或以内联方式提供的证书进行应用程序身份验证
* 使用之前为 Kusto 颁发的现有 AAD 令牌进行应用程序身份验证
* 使用为另一资源颁发的 AAD 令牌进行用户或应用程序身份验证，前提是该资源与 Kusto 之间存在信任关系

有关指导和示例，请查看 [Kusto 连接字符串](../../api/connection-strings/kusto.md)参考。

## <a name="user-authentication"></a>用户身份验证

当用户将凭据提交给 AAD（或某个与 AAD 联合的标识提供者，例如 ADFS）时，系统会进行用户身份验证，并会返回一个可以提交给 Kusto 服务的安全令牌。 Kusto 服务不关注安全令牌是如何获得的，它关注的是令牌是否有效，以及 AAD（或进行联合身份验证的 IdP）在其中放置了什么信息。

在客户端，Kusto 支持交互式身份验证，即通过 AAD 客户端库 ADAL 或类似的代码请求用户输入凭据。 它还支持基于令牌的身份验证，即通过使用 Kusto 的应用程序获取并提交有效的用户令牌。 最后，它还支持这样一种方案：可以通过使用 Kusto 的应用程序获得其他服务（不是 Kusto）的有效用户令牌，前提是该服务资源与 Kusto 之间存在信任关系。

请参阅 [Kusto 连接字符串](../../api/connection-strings/kusto.md)，详细了解如何使用 Kusto 客户端库，以及如何使用 AAD 向 Kusto 进行身份验证。

## <a name="application-authentication"></a>应用程序身份验证

如果请求不与特定用户相关联，或者没有用户输入凭据，则可使用 AAD 应用程序身份验证流。 在该流中，应用程序通过提供某种机密信息向 AAD（或进行联合身份验证的 IdP）进行身份验证。 以下方案受各种 Kusto 客户端的支持：

* 使用安装在本地的 X.509v2 证书进行的应用程序身份验证。
* 使用作为字节流提供给客户端库的 X.509v2 证书进行的应用程序身份验证。
* 使用 AAD 应用程序 ID 和 AAD 应用程序密钥进行的应用程序身份验证（相当于应用程序的用户名/密码身份验证）。
* 使用以前获得的有效的 AAD 令牌（颁发给 Kusto）进行的应用程序身份验证。
* 使用以前获得的颁发给其他资源的有效 AAD 令牌进行的应用程序身份验证，前提是该资源与 Kusto 之间存在信任关系。

## <a name="aad-server-application-permissions"></a>AAD 服务器应用程序权限

通常情况下，AAD 服务器应用程序可以定义多个权限（例如，只读权限和读写权限），且 AAD 客户端应用程序可以确定它在请求授权令牌时所需的权限。 在令牌获取的过程中，系统会要求用户授权 AAD 客户端应用程序代表用户执行操作，以获得这些权限。 如果用户同意，则这些权限将在颁发给 AAD 客户端应用程序的令牌的范围声明中列出。



AAD 客户端应用程序配置为从用户（AAD 称之为“资源所有者”）请求“访问 Kusto”权限。

## <a name="kusto-client-sdk-as-an-aad-client-application"></a>Kusto 客户端 SDK 充当 AAD 客户端应用程序

Kusto 客户端库在通过调用 ADAL (the AAD client library) 获取与 Kusto 通信所需的令牌时，会提供以下信息：

1. 从调用方收到的 AAD 租户
2. AAD 客户端应用程序 ID
3. AAD 客户端资源 ID
4. AAD ReplyUrl（身份验证成功完成后 AAD 服务将重定向到的 URL；ADAL 随后会捕获此重定向并从中提取授权代码）。
5. 群集 URI（“https://Cluster-and-region.kusto.chinacloudapi.cn”）。

ADAL 返回给 Kusto 客户端库的令牌以 Kusto AAD 服务器应用程序作为受众，以“访问 Kusto”权限作为范围。

## <a name="authenticating-with-aad-programmatically"></a>以编程方式通过 AAD 进行身份验证

以下文章介绍了如何以编程方式通过 AAD 向 Kusto 进行身份验证：

* [如何执行 AAD 身份验证](./how-to-authenticate-with-aad.md)
