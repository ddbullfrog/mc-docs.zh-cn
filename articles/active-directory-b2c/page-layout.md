---
title: 页面布局版本
titleSuffix: Azure AD B2C
description: 有关自定义策略中 UI 自定义的页面布局版本历史记录。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/02/2020
ms.author: v-junlch
ms.subservice: B2C
ms.openlocfilehash: 1a03ba86432e54d2407430505c7a27fdaa5ac22b
ms.sourcegitcommit: 2eb5a2f53b4b73b88877e962689a47d903482c18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89413315"
---
# <a name="page-layout-versions"></a>页面布局版本

页面布局包会定期更新，其中包括页面元素的修复和改进。 以下更改日志详述了在每个版本中引入的更改。

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

## <a name="self-asserted-page-selfasserted"></a>自断言页面 (selfasserted)

**2.1.0**

- 本地化和辅助功能修补程序。

**2.0.0**

- 在自定义策略中添加了对[显示控件](display-controls.md)的支持。

**1.2.0**

- “用户名/电子邮件”和“密码”字段现在使用 `form` HTML 元素，以允许 Microsoft Edge 和 Internet Explorer (IE) 正确保存此信息。
- 添加了可配置的用户输入验证延迟，以改善用户体验。
- 辅助功能修复
- 现在可以在 HTML 标记中添加 `data-preload="true"` 特性来控制 CSS 和 JavaScript 的加载顺序。
  - 在加载 HTML 模板的同时加载链接的 CSS 文件，这样它就不会在加载文件之间“闪烁”。
  - 控制在页面加载前提取和执行 `script` 标记的顺序。
- 电子邮件字段现在为 `type=email`，移动键盘将提供正确的建议
- 对 Chrome 转换的支持

**1.1.0**

- 删除了取消警报
- 错误元素的 CSS 类
- 显示/隐藏错误逻辑得到了改进
- 删除了默认的 CSS

**1.0.0**

- 初始版本

## <a name="unified-sign-in-sign-up-page-with-password-reset-link-unifiedssp"></a>带有密码重置链接的统一登录注册页面 (unifiedssp)

**2.1.0**

- 添加了对多个注册链接的支持。
- 根据策略中定义的谓词规则添加了对用户输入验证的支持。

**1.2.0**

- “用户名/电子邮件”和“密码”字段现在使用 `form` HTML 元素，以允许 Microsoft Edge 和 Internet Explorer (IE) 正确保存此信息。
- 辅助功能修复
- 现在可以在 HTML 标记中添加 `data-preload="true"` 特性来控制 CSS 和 JavaScript 的加载顺序。
  - 在加载 HTML 模板的同时加载链接的 CSS 文件，这样它就不会在加载文件之间“闪烁”。
  - 控制在页面加载前提取和执行 `script` 标记的顺序。
- 电子邮件字段现在为 `type=email`，移动键盘将提供正确的建议
- 对 Chrome 转换的支持

**1.1.0**

- 添加了“使我保持登录状态”(KMSI) 控件

**1.0.0**

- 初始版本

## <a name="mfa-page-multifactor"></a>MFA 页面 (multifactor)

**1.2.1**

- 默认模板上的辅助功能修复

**1.2.0**

- 辅助功能修复
- 现在可以在 HTML 标记中添加 `data-preload="true"` 特性来控制 CSS 和 JavaScript 的加载顺序。
  - 在加载 HTML 模板的同时加载链接的 CSS 文件，这样它就不会在加载文件之间“闪烁”。
  - 控制在页面加载前提取和执行 `script` 标记的顺序。
- 电子邮件字段现在为 `type=email`，移动键盘将提供正确的建议
- 对 Chrome 转换的支持

**1.1.0**

- 删除了“确认代码”按钮
- 代码的输入字段现在只接受最多六 (6) 个字符的输入
- 当用户输入 6 位代码时，该页面会自动尝试验证输入的代码，不需用户单击任何按钮
- 如果代码错误，则会自动清除输入字段
- 如果三 (3) 次尝试都输入了不正确的代码，B2C 会将一个错误发送回信赖方
- 辅助功能修复
- 删除了默认的 CSS

**1.0.0**

- 初始版本

## <a name="exception-page-globalexception"></a>异常页 (globalexception)

**1.2.0**

- 辅助功能修复
- 现在可以在 HTML 标记中添加 `data-preload="true"` 特性来控制 CSS 和 JavaScript 的加载顺序。
  - 在加载 HTML 模板的同时加载链接的 CSS 文件，这样它就不会在加载文件之间“闪烁”。
  - 控制在页面加载前提取和执行 `script` 标记的顺序。
- 电子邮件字段现在为 `type=email`，移动键盘将提供正确的建议
- 对 Chrome 转换的支持

**1.1.0**

- 辅助功能修复
- 删除了当策略中没有协定时会出现的默认消息
- 删除了默认的 CSS

**1.0.0**

- 初始版本

## <a name="other-pages-providerselection-claimsconsent-unifiedssd"></a>其他页（ProviderSelection、ClaimsConsent、UnifiedSSD）

**1.2.0**

- 辅助功能修复
- 现在可以在 HTML 标记中添加 `data-preload="true"` 特性来控制 CSS 和 JavaScript 的加载顺序。
  - 在加载 HTML 模板的同时加载链接的 CSS 文件，这样它就不会在加载文件之间“闪烁”。
  - 控制在页面加载前提取和执行 `script` 标记的顺序。
- 电子邮件字段现在为 `type=email`，移动键盘将提供正确的建议
- 对 Chrome 转换的支持

**1.0.0**

- 初始版本

<!-- Update_Description: wording update -->

