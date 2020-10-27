---
title: 配置 Azure 多重身份验证 | Microsoft Docs
description: 本文介绍如何在 Azure 门户中配置 Azure 多重身份验证设置
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
origin.date: 07/11/2018
ms.date: 11/30/2018
ms.author: v-junlch
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: a3c7d41a0b9bb9303a21890cb745115398440004
ms.sourcegitcommit: 753c74533aca0310dc7acb621cfff5b8993c1d20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211453"
---
# <a name="configure-azure-multi-factor-authentication-settings"></a>配置 Azure 多重身份验证设置

在启动并运行 Azure 多重身份验证后，可以参考本文进行管理。 本文涵盖了各种主题，可帮助你充分利用 Azure 多重身份验证。 并非所有 Azure 多重身份验证版本都提供全部功能。

| 功能 | 说明 | 
|:--- |:--- |
| [可选择验证方法](#selectable-verification-methods) |使用此功能，可选择用户可用的一列身份验证方法。 |

## <a name="selectable-verification-methods"></a>可选择验证方法

可使用“可选择验证方法”  功能，选择用户可使用的验证方法。 下表提供了这些方法的简要概述。

用户为其帐户注册 Azure 多重身份验证时，可从你启用的选项中选择其首选验证方法。 [为我的帐户设置双重验证帐户](../user-help/multi-factor-authentication-end-user-first-time.md)中提供了用户注册过程指导。

| 方法 | 说明 |
|:--- |:--- |
| 拨打电话 |拨打自动语音电话。 用户接听电话，并按电话键盘上的 # 进行身份验证。 此电话号码不会同步到本地 Active Directory。 |
| 向手机发送短信 |发送包含验证码的短信。 系统会提示用户在登录界面中输入验证代码。 此过程称为单向短信。 双向短信意味着用户必须短信回复一个特定代码。 已弃用双向短信，2018 年 11 月 14 日后不再受到支持。 届时，配置为使用双向短信的用户会自动切换到“电话呼叫”  验证。|
| 通过移动应用发送通知 |向手机或已注册设备发送推送通知。 用户将查看通知并选择 **验证** 来完成验证。 Microsoft Authenticator 应用可用于 [Windows Phone](https://go.microsoft.com/fwlink/?Linkid=825071)、[Android](https://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](https://go.microsoft.com/fwlink/?Linkid=825073)。 |
| 通过移动应用发送验证码 |Microsoft Authenticator 应用每隔 30 秒会生成一个新的 OATH 验证码。 用户将此验证码输入到登录界面中。 Microsoft Authenticator 应用可用于 [Windows Phone](https://go.microsoft.com/fwlink/?Linkid=825071)、[Android](https://go.microsoft.com/fwlink/?Linkid=825072) 和 [iOS](https://go.microsoft.com/fwlink/?Linkid=825073)。 |

### <a name="enable-and-disable-verification-methods"></a>启用和禁用可选择验证方法

1. 登录 [Azure 门户](https://portal.azure.cn)。
2. 在左侧，选择“Azure Active Directory” > “用户和组” > “所有用户”  。
3. 选择“多重身份验证”。
4. 在“多重身份验证”下，选择“服务设置”。
5. 在“服务设置”页上的“验证选项”下，选择/取消选择要向用户提供的方法。

   ![选择验证方法](./media/howto-mfa-mfasettings/authmethods.png)

6. 单击“保存” 。

<!-- Update_Description: link update -->