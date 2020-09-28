---
author: mmacy
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.date: 08/19/2020
ms.author: v-junlch
ms.openlocfilehash: 70b2917de526589086996aacd2cfbedd4c3a3f53
ms.sourcegitcommit: 7646936d018c4392e1c138d7e541681c4dfd9041
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2020
ms.locfileid: "91244843"
---
> [!IMPORTANT]
> MSAL.js 2.0 当前不支持配合使用 Azure AD B2C 和 PKCE 授权代码流。 目前，Azure AD B2C 建议使用 [教程：注册应用程序][implicit-flow]中所述的隐式流。 要跟踪此问题的进展，请参阅 [MSAL.js Wiki][msal-wiki]。

[github-issue]: https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/1795
[implicit-flow]: ../articles/active-directory-b2c/tutorial-register-applications.md
[msal-wiki]: https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/MSAL-browser-B2C-CORS-issue

