---
title: Azure 自动化帐户身份验证概述
description: 本文概述了 Azure 自动化帐户身份验证。
keywords: 自动化安全性, 安全的自动化; 自动化身份验证
services: automation
ms.subservice: process-automation
origin.date: 09/28/2020
ms.date: 11/02/2020
ms.topic: conceptual
ms.openlocfilehash: 7caad25cab5e9bb4ec9a1c6c29d473107deef475
ms.sourcegitcommit: ca5e5792f3c60aab406b7ddbd6f6fccc4280c57e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92750185"
---
# <a name="automation-account-authentication-overview"></a>自动化帐户身份验证概述

Azure 自动化让可以通过其他云提供程序（如 Amazon Web Services (AWS)）针对 Azure、本地中的资源来自动执行任务。 可以使用 Runbook 来自动完成任务；如果你有要在 Azure 外部管理的业务或操作过程，也可以使用混合 Runbook 辅助角色。 在上述任一环境中工作都需要权限，以使用所需的最小权限安全地访问资源。

本文介绍了 Azure 自动化支持的身份验证方案，以及如何基于你需要管理的一个或多个环境开始操作。

## <a name="automation-account"></a>自动化帐户

首次启动 Azure 自动化时，必须创建至少一个自动化帐户。 使用自动化帐户，你可以将自动化资源、Runbook、资产、配置与其他帐户的资源相隔离。 可以使用自动化帐户将资源隔离到独立的逻辑环境中。 例如，可以在开发环境中使用一个帐户，在生产环境中使用另一个帐户，并在本地环境中使用另一个账户。 Azure 自动化帐户不同于 Microsoft 帐户或在 Azure 订阅中创建的帐户。 有关创建自动化帐户的介绍，请参阅[创建自动化帐户](automation-quickstart-create-account.md)。

## <a name="automation-resources"></a>自动化资源

每个自动化帐户的自动化资源都与单个 Azure 区域相关联，但该帐户可以管理 Azure 订阅中的所有资源。 如果你的策略要求将数据和资源隔离到特定的区域，则这是在不同区域中创建自动化帐户的主要原因。

在 Azure 自动化中使用 Azure 资源管理器和 PowerShell cmdlet 针对资源创建的所有任务必须使用基于 Azure Active Directory (Azure AD) 组织标识凭据的身份验证向 Azure 进行身份验证。

## <a name="run-as-accounts"></a>运行方式帐户

Azure 自动化中的运行方式帐户提供的身份验证适用于管理 Azure 资源管理器资源或在经典部署模型上部署的资源。 Azure 自动化中有两种类型的运行方式帐户：

* Azure 运行方式帐户
* Azure 经典运行方式帐户

若要详细了解这两种部署模型，请参阅[资源管理器部署和经典部署](../azure-resource-manager/management/deployment-models.md)。

### <a name="run-as-account"></a>运行方式帐户

Azure 运行方式帐户基于 Azure 的 Azure 资源管理器部署和管理服务来管理 Azure 资源。

创建运行方式帐户时，它会执行以下任务：

* 创建使用自签名证书的 Azure AD 应用程序，在 Azure AD 中为此应用程序创建服务主体帐户，并在当前订阅中为此帐户分配[参与者](../role-based-access-control/built-in-roles.md#contributor)角色。 可将证书设置更改为“所有者”或其他任何角色。 有关详细信息，请参阅 [Azure 自动化中基于角色的访问控制](automation-role-based-access-control.md)。

* 在指定的自动化帐户中创建名为 `AzureRunAsCertificate` 的自动化证书资产。 该证书资产保存 Azure AD 应用程序使用的证书私钥。

* 在指定的自动化帐户中创建名为 `AzureRunAsConnection` 的自动化连接资产。 该连接资产保存应用程序 ID、租户 ID、订阅 ID 和证书指纹。

### <a name="azure-classic-run-as-account"></a>Azure 经典运行方式帐户

Azure 经典运行方式帐户根据经典部署模型管理 Azure 经典资源。 只有订阅的共同管理员才能创建或续订这种类型的运行方式帐户。

创建 Azure 经典运行方式帐户时，它会执行以下任务。

* 在订阅中创建管理证书。

* 在指定的自动化帐户中创建名为 `AzureClassicRunAsCertificate` 的自动化证书资产。 该证书资产保存管理证书使用的证书私钥。

* 在指定的自动化帐户中创建名为 `AzureClassicRunAsConnection` 的自动化连接资产。 该连接资产保存订阅名称、订阅 ID 和证书资产名称。

>[!NOTE]
>创建自动化帐户时，默认情况下不会同时创建 Azure 经典运行方式帐户。 此帐户是按照[管理运行方式帐户](manage-runas-account.md#create-a-run-as-account-in-azure-portal)一文所述步骤单独创建的。

## <a name="service-principal-for-run-as-account"></a>运行方式帐户的服务主体

默认情况下，运行方式帐户的服务主体对 Azure AD 没有读取权限。 如果你希望添加读取或管理 Azure AD 的权限，需要在“API 权限”下对服务主体授予该权限。 若要了解详细信息，请参阅[添加用于访问 Web API 的权限](../active-directory/develop/quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api)。

## <a name="role-based-access-control"></a>基于角色的访问控制

基于角色的访问控制在 Azure 资源管理器中可用，用于向 Azure AD 用户帐户和运行方式帐户授予允许的操作，并对服务主体进行身份验证。 请阅读 [Azure 自动化中基于角色的访问控制](automation-role-based-access-control.md)一文，详细了解如何开发自动化权限管理模型。

## <a name="runbook-authentication-with-hybrid-runbook-worker"></a>使用混合 Runbook 辅助角色的 Runbook 身份验证

在数据中心的混合 Runbook 辅助角色上运行的或针对其他云环境（如 AWS）中的计算服务运行的 Runbook 不能使用通常用于向 Azure 资源进行Runbook 身份验证的方法。 这是因为这些资源在 Azure 外部运行，因此，它们需要在自动化中定义自己的安全凭据，以便向需要在本地访问的资源进行身份验证。 有关使用 Runbook 辅助角色的 Runbook 身份验证的详细信息，请参阅[在混合 Runbook 辅助角色上运行 Runbook](automation-hrw-run-runbooks.md)。

对于在 Azure VM 上使用混合 Runbook 辅助角色的 Runbook，可以通过[使用托管标识的 Runbook 身份验证](automation-hrw-run-runbooks.md#runbook-auth-managed-identities)而不是运行方式帐户来向 Azure 资源进行身份验证。

## <a name="next-steps"></a>后续步骤

* 若要从 Azure 门户创建自动化帐户，请参阅[创建独立的 Azure 自动化帐户](automation-create-standalone-account.md)。
* 如果你喜欢使用模板创建帐户，请参阅[使用 Azure 资源管理器模板创建自动化帐户](quickstart-create-automation-account-template.md)。
