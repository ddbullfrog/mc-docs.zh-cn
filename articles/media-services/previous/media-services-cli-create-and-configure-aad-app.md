---
title: 使用 Azure CLI 创建 Azure AD 应用并配置为访问 Azure 媒体服务 API | Microsoft Docs
description: 本主题展示了如何使用 Azure CLI 创建 Azure AD 应用，并将它配置为访问 Azure 媒体服务 API。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 08/26/2019
ms.date: 09/28/2019
ms.author: v-jay
ms.openlocfilehash: 4e54bd4559b044760e1e50e5773ab7b914338988
ms.sourcegitcommit: 7ad3bfc931ef1be197b8de2c061443be1cf732ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91245691"
---
# <a name="use-azure-cli-to-create-an-azure-ad-app-and-configure-it-to-access-media-services-api"></a>使用 Azure CLI 创建 Azure AD 应用并将其配置为访问媒体服务 API

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> 不会向媒体服务 v2 添加任何新特性或新功能。 <br/>查看最新版本：[媒体服务 v3](../latest/index.yml)。 另请参阅[从 v2 到 v3 的迁移指南](../latest/migrate-from-v2-to-v3.md)

本主题展示了如何使用 Azure CLI 创建 Azure Active Directory (Azure AD) 应用程序和服务主体，以便访问 Azure 媒体服务资源。 

## <a name="prerequisites"></a>必备条件

- 一个 Azure 帐户。 有关详细信息，请参阅 [Azure 试用](https://www.azure.cn/pricing/1rmb-trial/)。
- 一个媒体服务帐户。 有关详细信息，请参阅[利用 Azure 门户创建 Azure 媒体服务帐户](media-services-portal-create-account.md)。

## <a name="use-the-powershell"></a>使用 PowerShell

登录 [Azure 门户](https://portal.azure.cn/)。

## <a name="create-an-azure-ad-app-and-configure-access-to-the-media-account-with-azure-cli"></a>使用 Azure CLI 创建 Azure AD 应用并配置为访问媒体帐户
 
```azurecli
az login
az ad sp create-for-rbac --name <appName> 
az role assignment create --assignee < user/app id> --role Contributor --scope <subscription/subscription id>
```

例如：

```azurecli
az role assignment create --assignee a3e068fa-f739-44e5-ba4d-ad57866e25a1 --role Contributor --scope /subscriptions/0b65e280-7917-4874-9fed-1307f2615ea2/resourceGroups/Default-AzureBatch-SouthCentralUS/providers/microsoft.media/mediaservices/sbbash
```

在此示例中，作用域  是媒体服务帐户的完整资源路径。 不过，作用域  可以为任意级别。

例如，作用域可以为下列级别之一：
 
* 订阅  级别。
* 资源组  级别。
* 资源  级别（例如，媒体帐户）。

有关详细信息，请参阅[使用 Azure CLI 创建 Azure 服务主体](https://docs.azure.cn/cli/create-an-azure-service-principal-azure-cli)

另请参阅[使用 Azure 命令行接口管理基于角色的访问控制](../../role-based-access-control/role-assignments-cli.md)。 

## <a name="next-steps"></a>后续步骤

开始[将文件上传到帐户](media-services-portal-upload-files.md)。