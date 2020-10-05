---
title: Azure AD Connect 同步：目录扩展 | Microsoft Docs
description: 本主题介绍 Azure AD Connect 中的目录扩展功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 995ee876-4415-4bb0-a258-cca3cbb02193
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2020
ms.subservice: hybrid
ms.author: v-junlch
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59d5feb2129454db302861fa75b6b470e1abac1f
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245240"
---
# <a name="azure-ad-connect-sync-directory-extensions"></a>Azure AD Connect 同步：目录扩展
通过目录扩展，可以使用本地 Active Directory 中的属性扩展 Azure AD 中的架构。 此功能允许使用继续在本地管理的属性来构建 LOB 应用。 可通过[扩展](https://docs.microsoft.com/graph/extensibility-overview)使用这些属性。 

目前，没有任何 Microsoft 365 工作负荷使用这些属性。

## <a name="customize-which-attributes-to-synchronize-with-azure-ad"></a>自定义要与 Azure AD 同步的属性

在安装向导的自定义设置路径中配置要同步的其他属性。

> [!NOTE]
> 在 1.2.65.0 之前的 Azure AD Connect 版本中，“可用属性”搜索框区分大小写。

![架构扩展向导](./media/how-to-connect-sync-feature-directory-extensions/extension2.png)  

安装显示以下属性，它们是有效的候选项：

* “用户”和“组”对象类型
* 单值属性：String、Boolean、Integer、Binary
* 多值属性：String、Binary


>[!NOTE]
> 虽然 Azure AD Connect 支持将多值 Active Directory 属性作为多值目录扩展同步到 Azure AD，但目前无法检索/使用在多值目录扩展属性中上传的数据。

属性列表是从安装 Azure AD Connect 期间创建的架构缓存中读取的。 如果已使用附加属性扩展了 Active Directory 架构，则必须[刷新架构](how-to-connect-installation-wizard.md#refresh-directory-schema)，然后这些新属性才可见。

Azure AD 中的对象最多可以有 100 个目录扩展属性。 最大长度为 250 个字符。 如果属性值更长，则同步引擎会将其截断。

## <a name="configuration-changes-in-azure-ad-made-by-the-wizard"></a>向导在 Azure AD 中进行的配置更改

在安装 Azure AD Connect 期间，会注册可以使用这些属性的应用程序。 可以在 Azure 门户中看到此应用程序。 其名称始终为“租户架构扩展应用”  。

![架构扩展应用](./media/how-to-connect-sync-feature-directory-extensions/extension3new.png)

请确保选择“所有应用程序”  以查看此应用。

这些属性的前面带有扩展名 \_{ApplicationId}\_ 前缀。 对于 Azure AD 租户中的所有属性，ApplicationId 具有相同的值。 本主题中的所有其他方案都需要此值。

## <a name="viewing-attributes-using-the-microsoft-graph-api"></a>使用 Microsoft Graph API 查看属性

通过使用 [Microsoft Graph 浏览器](https://developer.microsoft.com/zh-cn/graph/graph-explorer-china)，这些属性现在可以通过 Microsoft Graph API 获得。

>[!NOTE]
> 在 Microsoft Graph API 中，你需要请求要返回的属性。 按以下所示显式选择属性：`https://microsoftgraph.chinacloudapi.cn/beta/users/abbie.spencer@fabrikamonline.com?$select=extension_9d98ed114c4840d298fad781915f27e4_employeeID,extension_9d98ed114c4840d298fad781915f27e4_division`。
>
> 有关详细信息，请参阅 [Microsoft Graph: Use query parameters](https://docs.microsoft.com/graph/query-parameters#select-parameter)（Microsoft Graph：使用查询参数）。

## <a name="next-steps"></a>后续步骤
了解有关 [Azure AD Connect 同步](how-to-connect-sync-whatis.md)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](whatis-hybrid-identity.md)的详细信息。

<!-- Update_Description: wording update -->