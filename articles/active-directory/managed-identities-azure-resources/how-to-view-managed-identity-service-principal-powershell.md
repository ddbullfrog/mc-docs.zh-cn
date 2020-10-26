---
title: 使用 PowerShell 查看托管标识的服务主体 - Azure AD
description: 使用 PowerShell 查看托管标识的服务主体的分步说明。
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/12/2020
ms.author: v-junlch
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c0a03c2e5ad9717f1898bc758a82da9b7e1897e1
ms.sourcegitcommit: 4d06a5e0f48472f5eadd731e43afb1e9fbba5787
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041606"
---
# <a name="view-the-service-principal-of-a-managed-identity-using-powershell"></a>使用 PowerShell 查看托管标识的服务主体

Azure 资源的托管标识在 Azure Active Directory 中为 Azure 服务提供了一个自动托管标识。 此标识可用于通过支持 Azure AD 身份验证的任何服务的身份验证，这样就无需在代码中插入凭据了。 

本文介绍如何使用 PowerShell 查看托管标识的服务主体。

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>必备条件

- 如果不熟悉 Azure 资源的托管标识，请查阅[概述部分](overview.md)。
- 如果还没有 Azure 帐户，请注册[试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。
- 在[虚拟机](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity)或[应用程序](../../app-service/overview-managed-identity.md#add-a-system-assigned-identity)上启用系统分配的标识。
- 若要运行示例脚本，可以通过安装最新版的 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps) 以在本地运行脚本，然后使用 `Connect-AzAccount -Environment AzureChinaCloud` 登录到 Azure。

## <a name="view-the-service-principal"></a>查看服务主体

以下命令演示了如何查看启用了系统分配的标识的 VM 或应用程序的服务主体。 将 `<VM or application name>` 替换为自己的值。

```azurepowershell
Get-AzADServicePrincipal -DisplayName <VM or application name>
```

## <a name="next-steps"></a>后续步骤

有关使用 PowerShell 查看 Azure AD 服务主体的详细信息，请参阅 [Get-AzADServicePrincipal](https://docs.microsoft.com/powershell/module/az.resources/get-azadserviceprincipal)。

