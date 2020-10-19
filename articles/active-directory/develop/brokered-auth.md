---
title: Android 中的中介身份验证 | Azure
titleSuffix: Microsoft identity platform
description: Microsoft 标识平台中适用于 Android 的中介身份验证和授权概述
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/30/2020
ms.author: v-junlch
ms.custom: aaddev
ms.reviewer: shoatman, hahamil, brianmel
ms.openlocfilehash: 35a97f4e6903e4bd305eea0cc7f8d5b4f95e054e
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937184"
---
# <a name="brokered-authentication-in-android"></a>Android 中的中介身份验证

必须使用 Microsoft 的某个身份验证中介，来参与设备范围的单一登录 (SSO)，并保证符合组织条件访问策略的要求。 与中介集成可提供以下优势：

- 设备单一登录
- 对以下功能进行条件访问：
  - Intune 应用保护
  - 设备注册 (Workplace Join)
  - 移动设备管理
- 设备范围的帐户管理
  -  通过 Android AccountManager 和帐户设置
  - “工作帐户”- 自定义帐户类型

在 Android 上，Microsoft 身份验证中介是 Microsoft Authenticator 应用和 Intune 公司门户随附的一个组件。

下图演示了应用、Microsoft 身份验证库 (MSAL) 与 Microsoft 身份验证中介之间的关系。

![关系图显示了应用程序如何与 MSAL、中介应用以及 Android 帐户管理器进行关联。](./media/brokered-auth/brokered-deployment-diagram.png)

## <a name="installing-apps-that-host-a-broker"></a>安装托管中介的应用

设备所有者随时可以从应用商店（通常是 Google Play 商店）安装中介托管应用。 但是，某些 API（资源）受条件访问策略的保护，这些策略要求设备：

- 已注册（已加入工作区），和/或
- 已在设备管理中注册，或
- 已在 Intune 应用保护中注册

如果某个设备上尚未安装中介应用，当应用尝试以交互方式获取令牌时，MSAL 会立即指示用户安装一个中介应用。 然后，应用需要引导用户完成相关步骤，使设备符合所需的策略。

## <a name="effects-of-installing-and-uninstalling-a-broker"></a>安装和卸载中介的影响

### <a name="when-a-broker-is-installed"></a>已安装中介时

在设备上安装中介后，所有后续交互式令牌请求（对 `acquireToken()` 的调用）将由该中介处理，而不是由 MSAL 在本地处理。 以前提供给 MSAL 的任何 SSO 状态不会提供给中介。 因此，用户需要重新进行身份验证，或者从设备已知的现有帐户列表中选择一个帐户。

安装中介无需用户再次登录。 仅当用户需要解决 `MsalUiRequiredException` 时，下一个请求才会发送到中介。 引发 `MsalUiRequiredException` 的原因有很多，需要以交互方式解决。 例如：

- 用户更改了与其帐户关联的密码。
- 用户的帐户不再符合条件访问策略。
- 用户撤销了应用与其帐户关联的许可。

#### <a name="multiple-brokers"></a>多个中介

如果设备上安装了多个中介，则首先安装的代理始终是活动中介。 仅单个中介可以在设备上处于活动状态。

### <a name="when-a-broker-is-uninstalled"></a>已卸载中介时

如果只安装了一个中介托管应用，后来删除了该应用，则用户需要再次登录。 卸载活动中介会从设备中删除相应的帐户和关联的令牌。

如果 Intune 公司门户已安装并作为活动中介运行，此外还安装了 Microsoft Authenticator，则在卸载 Intune 公司门户（活动中介）后，用户需要再次登录。 一旦用户再次登录，Microsoft Authenticator 应用就会成为活动中介。

## <a name="integrating-with-a-broker"></a>与中介集成

### <a name="generating-a-redirect-uri-for-a-broker"></a>为中介生成重定向 URI

必须注册与中介兼容的重定向 URI。 中介的重定向 URI 应该包含应用的包名称，以及应用签名的 base64 编码表示形式。

重定向 URI 的格式为：`msauth://<yourpackagename>/<base64urlencodedsignature>`

使用应用的签名密钥，你可以使用 [keytool](https://manpages.debian.org/buster/openjdk-11-jre-headless/keytool.1.en.html) 生成 Base64 编码的签名哈希，然后使用 Azure 门户通过该哈希生成重定向 URI。

Linux 和 macOS：

```bash
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```

Windows:

```powershell
keytool -exportcert -alias androiddebugkey -keystore %HOMEPATH%\.android\debug.keystore | openssl sha1 -binary | openssl base64
```

使用 keytool 生成签名哈希后，请使用 Azure 门户生成重定向 URI：

1. 登录到 [Azure 门户](https://portal.azure.cn)，并选择“应用注册”中的 Android 应用。
1. 选择“身份验证” > “添加平台” > “Android”  。
1. 在“配置 Android 应用”窗格打开时，输入你之前生成的“签名哈希”，然后输入“包名称”  。
1. 选择“配置”按钮。

Azure 门户会为你生成重定向 URI，并将其显示在“Android 配置”窗格的“重定向 URI”字段中 。

有关对应用进行签名的详细信息，请参阅 Android Studio 用户指南中的[对应用进行签名](https://developer.android.com/studio/publish/app-signing)。

> [!IMPORTANT]
> 对于生产版本的应用，请使用生产签名密钥。

### <a name="configure-msal-to-use-a-broker"></a>将 MSAL 配置为使用中介

若要在应用中使用中介，必须证明你已配置中介重定向。 例如，通过在 MSAL 配置文件中包含以下设置，来包含中介启用的重定向 URI，并指示已注册该中介：

```json
"redirect_uri" : "<yourbrokerredirecturi>",
"broker_redirect_uri_registered": true
```

### <a name="broker-related-exceptions"></a>中介相关的异常

MSAL 通过两种方式来与中介通信：

- 中介绑定服务
- Android AccountManager

MSAL 首先使用中介绑定服务，因为调用此服务不需要任何 Android 权限。 如果绑定到绑定服务失败，MSAL 将使用 Android AccountManager API。 仅当已为应用授予 `"READ_CONTACTS"` 权限时，MSAL 才使用此 API。

如果收到 `MsalClientException` 和错误代码 `"BROKER_BIND_FAILURE"`，可以采取两种做法：

- 要求用户禁用 Microsoft Authenticator 应用和 Intune 公司门户的超级优化。
- 要求用户授予 `"READ_CONTACTS"` 权限

## <a name="verifying-broker-integration"></a>验证中介集成

虽然可能无法立即弄清楚中介集成是否正在运行，但你可以使用以下步骤进行检查：

1. 在 Android 设备上，使用中介完成某个请求。
1. 在 Android 设备上的设置中，查找与进行身份验证时所使用的帐户相对应的新建帐户。 该帐户的类型应为“工作帐户”。

如果要重复测试，可以从设置中删除该帐户。

## <a name="next-steps"></a>后续步骤

使用[适用于 Android 设备的共享设备模式](msal-android-shared-devices.md)，可以对 Android 设备进行配置，使其可由多名员工轻松共享。

