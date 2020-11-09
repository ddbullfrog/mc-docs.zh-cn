---
title: 管理 Azure 自动化运行方式帐户
description: 本文介绍如何使用 PowerShell 或从 Azure 门户管理运行方式帐户。
services: automation
ms.subservice: shared-capabilities
origin.date: 09/28/2020
ms.date: 11/02/2020
ms.topic: conceptual
ms.openlocfilehash: f946d1ca5eb2c0368834a478559bdfa3e1de3f6b
ms.sourcegitcommit: ca5e5792f3c60aab406b7ddbd6f6fccc4280c57e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92750198"
---
# <a name="manage-an-azure-automation-run-as-account"></a>管理 Azure 自动化运行方式帐户

Azure 自动化中的运行方式帐户提供身份验证，以使用自动化 runbook 和其他自动化功能管理 Azure 资源管理器或 Azure 经典部署模型上的资源。 本文提供有关如何管理运行方式帐户或经典运行方式帐户的指导。

若要详细了解 Azure 自动化帐户身份验证以及有关流程自动化方案的指导，请参阅[自动化帐户身份验证概述](automation-security-overview.md)。

## <a name="run-as-account-permissions"></a><a name="permissions"></a>运行方式帐户的权限

本部分定义普通运行方式帐户和经典运行方式帐户的权限。

若要创建或更新运行方式帐户，必须拥有特定的特权和权限。 Azure Active Directory 中的应用程序管理员和订阅中的所有者都可以完成所有任务。 下表显示了在实施职责分离的情况下，所需的任务、等效 cmdlet 和权限的列表：

|任务|Cmdlet  |最低权限  |设置权限的位置|
|---|---------|---------|---|
|创建 Azure AD 应用程序|[New-AzADApplication](https://docs.microsoft.com/powershell/module/az.resources/new-azadapplication)     | 应用程序开发人员角色<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>主页 > Azure AD > 应用注册 |
|将凭据添加到应用程序。|[New-AzADAppCredential](https://docs.microsoft.com/powershell/module/az.resources/new-azadappcredential)     | 应用程序管理员或全局管理员<sup>1</sup>         |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>主页 > Azure AD > 应用注册|
|创建和获取 Azure AD 服务主体|[New-AzADServicePrincipal](https://docs.microsoft.com/powershell/module/az.resources/new-azadserviceprincipal)</br>[Get-AzADServicePrincipal](https://docs.microsoft.com/powershell/module/az.resources/get-azadserviceprincipal)     | 应用程序管理员或全局管理员<sup>1</sup>        |[Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)</br>主页 > Azure AD > 应用注册|
|分配或获取指定主体的 Azure 角色|[New-AzRoleAssignment](https://docs.microsoft.com/powershell/module/az.resources/new-azroleassignment)</br>[Get-AzRoleAssignment](https://docs.microsoft.com/powershell/module/Az.Resources/Get-AzRoleAssignment)      | 用户访问管理员或所有者，或具有以下权限：</br></br><code>Microsoft.Authorization/Operations/read</br>Microsoft.Authorization/permissions/read</br>Microsoft.Authorization/roleDefinitions/read</br>Microsoft.Authorization/roleAssignments/write</br>Microsoft.Authorization/roleAssignments/read</br>Microsoft.Authorization/roleAssignments/delete</code></br></br> | [订阅](../role-based-access-control/role-assignments-portal.md)</br>主页 > 订阅 > \<subscription name\> - 访问控制 (IAM)|
|创建或删除自动化证书|[New-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/New-AzAutomationCertificate)</br>[Remove-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/az.automation/remove-azautomationcertificate)     | 资源组中的参与者         |自动化帐户资源组|
|创建或删除自动化连接|[New-AzAutomationConnection](https://docs.microsoft.com/powershell/module/az.automation/new-azautomationconnection)</br>[Remove-AzAutomationConnection](https://docs.microsoft.com/powershell/module/az.automation/remove-azautomationconnection)|资源组中的参与者 |自动化帐户资源组|

<sup>1</sup> Azure AD 租户中的非管理员用户可以[注册 AD 应用程序](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app)，前提是 Azure AD 租户的“用户设置”页中的“用户可以注册应用程序”选项已设置为“是” 。 如果应用程序注册设置为“否”，则执行此操作的用户必须具有此表中定义的角色。

如果你在被添加到订阅的全局管理员角色之前不是订阅的 Active Directory 实例的成员，则会将你添加为来宾。 在这种情况下，“添加自动化帐户”页上会显示 `You do not have permissions to create…` 警告。

如果在分配全局管理员角色时，你是订阅的 Active Directory 实例的成员，则“添加自动化帐户”页面上还会显示 `You do not have permissions to create…` 警告。 在这种情况下，可以请求从订阅的 Active Directory 实例中删除，然后请求重新添加，以便成为 Active Directory 中的完整用户。

若要验证是否修正了生成该错误消息的原因：

1. 在 Azure 门户上的“Azure Active Directory”窗格中，选择“用户”。 
2. 选择“所有用户”。
3. 选择名称，然后选择“配置文件”。
4. 请确保用户配置文件下“用户类型”属性的值未设置为“来宾” 。

### <a name="permissions-required-to-create-or-manage-classic-run-as-accounts"></a><a name="permissions-classic"></a>创建或管理经典运行方式帐户所需的权限

若要配置或续订经典运行方式帐户，需要在订阅级别具有共同管理员角色。 若要详细了解有关经典订阅权限，请参阅 [Azure 经典订阅管理员](../role-based-access-control/classic-administrators.md#add-a-co-administrator)。

## <a name="create-a-run-as-account-in-azure-portal"></a>在 Azure 门户中创建运行方式帐户

请执行以下步骤，在 Azure 门户中更新 Azure 自动化帐户。 可以单独创建运行方式帐户和经典运行方式帐户。 如果不需管理经典资源，可以只创建 Azure 运行方式帐户。

1. 以订阅管理员角色成员和订阅共同管理员的帐户登录到 Azure 门户。

2. 搜索并选择“自动化帐户”。

3. 在“自动化帐户”页，选择列表中的自动化帐户。

4. 在左侧窗格中的“帐户设置”部分选择“运行方式帐户” 。

    :::image type="content" source="media/manage-runas-account/automation-account-properties-pane.png" alt-text="选择“运行方式帐户”选项。":::

5. 根据所需帐户，使用“+ Azure 运行方式帐户”或“+ Azure 经典运行方式帐户”窗格 。 查看概述信息后，单击“创建”。

    :::image type="content" source="media/manage-runas-account/automation-account-create-runas.png" alt-text="选择创建运行方式帐户的选项":::

6. 在 Azure 创建运行方式帐户时，可以在菜单的“通知”下面跟踪进度。 此外还显示一个横幅，指出正在创建帐户。 此过程可能需要几分钟才能完成。

## <a name="delete-a-run-as-or-classic-run-as-account"></a>删除运行方式帐户或经典运行方式帐户

本部分介绍如何删除运行方式帐户或经典运行方式帐户。 执行此操作时，将保留自动化帐户。 删除运行方式帐户后，可以在 Azure 门户中重新创建它。

1. 在 Azure 门户中，打开自动化帐户。

2. 在左侧窗格中，选择帐户设置部分中的“运行方式帐户”。

3. 在“运行方式帐户”属性页上，选择要删除的运行方式帐户或经典运行方式帐户。

4. 在所选帐户的“属性”窗格中单击“删除”。

   ![删除运行方式帐户](media/manage-runas-account/automation-account-delete-runas.png)

5. 帐户删除过程中，可以在菜单的“通知”下面跟踪进度。

## <a name="renew-a-self-signed-certificate"></a><a name="cert-renewal"></a>续订自签名证书

为运行方式帐户创建的自签名证书自创建日期算起的一年后过期。 在运行方式帐户过期之前的某个时间点，必须续订证书。 可以在该证书过期之前的任何时间续订。

续订自签名证书时，将保留当前的有效证书，以确保已排队等候或正在主动运行且使用运行方式帐户进行身份验证的任何 Runbook 不会受到负面影响。 该证书在过期之前将保持有效。

>[!NOTE]
>如果认为运行方式帐户已遭到入侵，可以删除该自签名证书然后重新创建。

>[!NOTE]
>如果已将运行方式帐户配置为使用企业证书颁发机构颁发的证书，并使用此选项来续订自签名证书选项，该企业证书会被自签名证书替换。

使用以下步骤来续订自签名证书。

1. 在 Azure 门户中，打开自动化帐户。

1. 选择帐户设置部分中的“运行方式帐户”。

    ![自动化帐户属性窗格](media/manage-runas-account/automation-account-properties-pane.png)

1. 在“运行方式帐户”属性页上，选择要为其续订证书的运行方式帐户或经典运行方式帐户。

1. 在所选帐户的“属性”窗格中，单击“续订证书”。

    ![续订运行方式帐户的证书](media/manage-runas-account/automation-account-renew-runas-certificate.png)

1. 证书续订过程中，可以在菜单的“通知”下面跟踪进度。

## <a name="limit-run-as-account-permissions"></a>限制运行方式帐户权限

若要针对 Azure 中的资源控制自动化的目标，可以运行 [Update-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug8) 脚本。 此脚本将更改现有运行方式帐户服务主体，以创建和使用自定义角色定义。 该角色具有除了 [Key Vault](../key-vault/index.yml) 之外的所有资源的权限。

>[!IMPORTANT]
>运行 Update-AutomationRunAsAccountRoleAssignments.ps1 脚本后，通过使用运行方式帐户访问 Key Vault 的 Runbook 将不再工作。 在运行该脚本之前，应查看帐户中的 Runbook，以便调用 Azure Key Vault。 若要实现从 Azure 自动化 Runbook 对 Key Vault 的访问，必须[将运行方式帐户添加到 Key Vault 的权限](#add-permissions-to-key-vault)。

如果需要进一步限制运行方式服务主体可执行的内容，可将其他资源类型添加到自定义角色定义的 `NotActions` 元素。 下面的示例限制对 `Microsoft.Compute/*` 的访问。 如果将此资源类型添加到角色定义的 `NotActions`，则该角色将不能访问任何计算资源。 若要详细了解角色定义，请参阅[了解 Azure 资源的角色定义](../role-based-access-control/role-definitions.md)。

```powershell
$roleDefinition = Get-AzRoleDefinition -Name 'Automation RunAs Contributor'
$roleDefinition.NotActions.Add("Microsoft.Compute/*")
$roleDefinition | Set-AzRoleDefinition
```

可确定运行方式帐户使用的服务主体是在参与者角色定义中还是在自定义角色定义中。

1. 转到自动化帐户，然后在帐户设置部分中选择“运行方式帐户”。
2. 选择“Azure 运行方式帐户”。
3. 选择“角色”以查找正在使用的角色定义。

:::image type="content" source="media/manage-runas-account/verify-role.png" alt-text="验证运行方式帐户角色。" lightbox="media/manage-runas-account/verify-role-expanded.png":::

还可以为多个订阅或自动化帐户确定运行方式帐户使用的角色定义。 为此，请使用 PowerShell 库中的 [Check-AutomationRunAsAccountRoleAssignments.ps1](https://aka.ms/AA5hug5) 脚本。

### <a name="add-permissions-to-key-vault"></a>添加 Key Vault 权限

可以允许 Azure 自动化验证 Key Vault 和运行方式帐户服务主体是否正在使用自定义角色定义。 必须具备以下条件：

* 授予 Key Vault 权限。
* 设置访问策略。

可以使用 PowerShell 库中的 [Extend-AutomationRunAsAccountRoleAssignmentToKeyVault.ps1](https://aka.ms/AA5hugb) 脚本，向运行方式帐户授予 Key Vault 权限。 有关设置对 Key Vault 的权限的详细信息，请参阅[分配 Key Vault 访问策略](../key-vault/general/assign-access-policy-powershell.md)。

## <a name="resolve-misconfiguration-issues-for-run-as-accounts"></a>解决运行方式帐户的错误配置问题

在初始设置期间，运行方式帐户或经典运行方式帐户所需的某些配置项可能已被删除或未正确创建。 错误配置的实例可能包括：

* 证书资产
* 连接资产
* 已从参与者角色中删除的运行方式帐户
* Azure AD 中的服务主体或应用程序

对于此类错误配置实例，自动化帐户将检测更改，并在该帐户的“运行方式帐户”属性窗格中显示“不完整”状态。

![不完整的运行方式帐户配置状态](media/manage-runas-account/automation-account-runas-config-incomplete.png)

选择该运行方式帐户时，该帐户的“属性”窗格中会显示以下错误消息：

```text
The Run As account is incomplete. Either one of these was deleted or not created - Azure Active Directory Application, Service Principal, Role, Automation Certificate asset, Automation Connect asset - or the Thumbprint is not identical between Certificate and Connection. Please delete and then re-create the Run As Account.
```

可通过删除并重新创建运行方式帐户来快速解决这些运行方式帐户问题。

## <a name="next-steps"></a>后续步骤

* [应用程序对象和服务主体对象](../active-directory/develop/app-objects-and-service-principals.md)。
* [Azure 云服务证书概述](../cloud-services/cloud-services-certs-create.md)。
