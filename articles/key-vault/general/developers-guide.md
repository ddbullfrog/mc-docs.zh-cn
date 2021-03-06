---
title: Azure 密钥保管库开发人员指南
description: 开发人员可以使用 Azure 密钥保管库来管理 Azure 环境中的加密密钥。
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
origin.date: 03/11/2020
ms.date: 09/15/2020
ms.author: v-tawe
ms.openlocfilehash: 614aa7e8572068e8301912b4386b75b60a28692f
ms.sourcegitcommit: 39410f3ed7bdeafa1099ba5e9ec314b4255766df
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/16/2020
ms.locfileid: "90678510"
---
# <a name="azure-key-vault-developers-guide"></a>Azure 密钥保管库开发人员指南

使用 Key Vault 可以从应用程序中安全地访问敏感信息：

- 无需自己编写代码即可保护密钥和机密信息，并且能够轻松地在应用程序中使用它们。
- 能够让客户拥有和管理其自己的密钥，因此可以专注于提供核心软件功能。 这样，应用程序便不会对客户的租户密钥和机密承担职责或潜在责任。
- 应用程序可以使用密钥进行签名和加密，不过使密钥管理与应用程序分开，可以使解决方案适用于地理分散的应用。
- 管理 Key Vault 证书。 有关详细信息，请参阅[证书](../certificates/about-certificates.md)

有关 Azure Key Vault 的更多常规信息，请参阅[什么是 Key Vault](overview.md)。

## <a name="public-previews"></a>公共预览版

我们会定期发布新 Key Vault 功能的公共预览版。 抢先试用这些版本，并通过反馈电子邮件地址 azurekeyvault@microsoft.com 将想法告诉我们。

## <a name="creating-and-managing-key-vaults"></a>创建和管理密钥保管库

虽然 Azure Key Vault 可用于安全存储凭据以及其他密钥和机密，但代码需要通过 Key Vault 的身份验证才能检索它们。 Azure 资源的托管标识为 Azure 服务提供了 Azure Active Directory (Azure AD) 中的自动托管标识，更巧妙地解决了这个问题。 此标识可用于通过支持 Azure AD 身份验证的任何服务（包括 Key Vault）的身份验证，这样就无需在代码中插入任何凭据了。 

有关 Azure 资源的托管标识的详细信息，请参阅[标识概述](../../active-directory/managed-identities-azure-resources/overview.md)。 若要详细了解如何使用 Azure AD，请参阅[将应用程序与 Azure Active Directory 集成](../../active-directory/develop/active-directory-integrating-applications.md)。

使用密钥保管库中的密钥、机密或证书前，请通过 CLI、PowerShell、资源管理器模板或 REST 创建和管理密钥保管库，如以下文章所述：

- [使用 CLI 创建和管理 Key Vault](quick-create-cli.md)
- [使用 PowerShell 创建和管理 Key Vault](quick-create-powershell.md)
- [使用 Azure 门户创建和管理 Key Vault](quick-create-portal.md)
- [使用 REST 创建和管理 Key Vault](https://docs.microsoft.com/rest/api/keyvault/vaults/createorupdate)

### <a name="set-and-retrieve-secrets"></a>设置和检索机密

- [使用 CLI 设置和检索机密](../secrets/quick-create-cli.md)
- [使用 PowerShell 设置和检索机密](../secrets/quick-create-powershell.md)
- [使用 Azure 门户设置和检索机密](../secrets/quick-create-portal.md)
- [REST 的机密操作](https://docs.microsoft.com/rest/api/keyvault/#secret-operations)
- [使用 Python 设置和检索机密](../secrets/quick-create-python.md)
- [使用 Java 设置和检索机密](../secrets/quick-create-java.md)
- [使用 Node.js 设置和检索机密](../secrets/quick-create-node.md)
- [使用 .NET (v4 SDK) 设置和检索机密](../secrets/quick-create-net.md)
- [通过 Azure Resource Manager 模板创建密钥保管库并添加机密](../secrets/quick-create-template.md)

### <a name="set-and-retrieve-keys"></a>设置和检索密钥

- [使用 CLI 设置和检索密钥](../keys/quick-create-cli.md)
- [使用 PowerShell 设置和检索密钥](../keys/quick-create-powershell.md)
- [使用 Azure 门户设置和检索密钥](../keys/quick-create-portal.md)
- [REST 的密钥操作](https://docs.microsoft.com/rest/api/keyvault/#key-operations)
- [使用 Python 设置和检索密钥](../secrets/quick-create-python.md)

### <a name="set-and-retrieve-certificates"></a>设置和检索证书
- [使用 CLI 设置和检索证书](../certificates/quick-create-cli.md)
- [使用 PowerShell 设置和检索证书](../certificates/quick-create-powershell.md)
- [使用 Azure 门户设置和检索证书](../certificates/quick-create-portal.md)
- [REST 的证书操作](https://docs.microsoft.com/rest/api/keyvault/#certificate-operations)
- [使用 Python 设置和检索证书](../certificates/quick-create-python.md)

## <a name="coding-with-key-vault"></a>使用密钥保管库进行编码

面向程序员的 Key Vault 管理系统包含多个接口。 此部分收录了所有语言和一些代码示例的链接。 

### <a name="supported-programming-and-scripting-languages"></a>支持的编程和脚本语言

#### <a name="rest"></a>REST

通过 REST 接口，可以访问所有 Key Vault 资源：保管库、密钥、机密等。 

[Key Vault REST API Reference](https://docs.microsoft.com/rest/api/keyvault/)（Key Vault REST API 参考）。 

#### <a name="net"></a>.NET

[适用于 Key Vault 的 .NET API 参考](https://docs.microsoft.com/dotnet/api/overview/key-vault?view=azure-dotnet )。

#### <a name="java"></a>Java

[Java SDK for Key Vault](https://docs.microsoft.com/java/api/overview/keyvault)（适用于 Key Vault 的 Java SDK）

#### <a name="nodejs"></a>Node.js

在 Node.js 中，Key Vault 管理 API 和 Key Vault 对象 API 相互独立。 下面的概述文章介绍了如何访问这两个 API。 

[用于 Node.js 的 Azure Key Vault 模块](https://docs.microsoft.com/javascript/api/overview/azure/key-vault-index?view=azure-node-latest)

#### <a name="python"></a>Python

[用于 Python 的 Azure Key Vault 库](https://docs.microsoft.com/python/api/overview/azure/key-vault-index?view=azure-python)

#### <a name="azure-cli"></a>Azure CLI

[适用于 Key Vault 的 Azure CLI](/cli/keyvault?view=azure-cli-latest)

#### <a name="azure-powershell"></a>Azure PowerShell 

[适用于 Key Vault 的 Azure PowerShell](https://docs.microsoft.com/powershell/module/az.keyvault/?view=azps-3.6.1#key_vault)

### <a name="code-examples"></a>代码示例

有关在应用程序中使用密钥保管库的完整示例，请参阅：

- [Azure Key Vault 代码示例](https://azure.microsoft.com/resources/samples/?service=key-vault) - Azure Key Vault 的代码示例。 

## <a name="how-tos"></a>操作方法

以下文章和方案提供了特定于任务的指导，方便用户使用 Azure Key Vault：

<!-- - [How to Generate and Transfer HSM-Protected Keys for Azure Key Vault](key-vault-hsm-protected-keys.md) - This will help you plan for, generate and then transfer your own HSM-protected keys to use with Azure Key Vault. -->

- [订阅移动后更改密钥保管库租户 ID](move-subscription.md) - 将 Azure 订阅从租户 A 移到租户 B 时，租户 B.中的主体（用户和应用程序）无法访问现有的密钥保管库。使用本指南解决此问题。
- [访问防火墙后面的密钥保管库](access-behind-firewall.md) - 若要访问密钥保管库，密钥保管库客户端应用程序需要能够访问多个终结点才能使用各种功能。
- [如何在部署期间传递安全值（如密码）](../../azure-resource-manager/templates/key-vault-parameter.md) - 需要在部署期间以参数形式传递安全值（例如密码）时，可以将该值存储为 Azure Key Vault 中的机密，并在其他资源管理模板中引用该值。
- [如何使用 Key Vault，以便通过 SQL Server 进行可扩展的密钥管理](https://msdn.microsoft.com/library/dn198405.aspx) - 适用于 Azure Key Vault 的 SQL Server 连接器允许 SQL Server 和 VM 中的 SQL 将 Azure Key Vault 服务用作可扩展密钥管理 (EKM) 提供程序，以便保护其针对应用程序链接的加密密钥；透明数据加密、备份加密和列级加密。
- [如何将 Key Vault 中的证书部署到 VM](https://blogs.technet.microsoft.com/kv/2015/07/14/deploy-certificates-to-vms-from-customer-managed-key-vault/) - 在 Azure 上的 VM 中运行的云应用程序需要一个证书。 现在，如何将此证书部署到此 VM 中？
- [通过 Key Vault 部署 Azure Web 应用证书]( https://blogs.msdn.microsoft.com/appserviceteam/2016/05/24/deploying-azure-web-app-certificate-through-key-vault/)提供有关部署作为[应用服务证书](https://azure.microsoft.com/blog/internals-of-app-service-certificate/)产品的一部分存储在 Key Vault 中的证书的分步说明。
- 分配访问策略（[CLI](assign-access-policy-cli.md) | [PowerShell](assign-access-policy-powershell.md) | [门户](assign-access-policy-portal.md)）。 Key Vault 最多支持 1024 个访问策略条目。 若要使用户保持在此限制范围内，请创建 Azure Active Directory 安全组，将所有关联的服务主体添加到该组，然后向该组授予对 Key Vault 的访问权限。
- 如需更多将 Key Vault 与 Azure 集成和结合使用的特定于任务的指导，请参阅 [Ryan Jones Azure Resource Manager template examples for Key Vault](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples)（针对 Key Vault 的 Ryan Jones Azure 资源管理器模板示例）。
- [如何将 Key Vault 软删除与 CLI 配合使用](soft-delete-cli.md)介绍了 Key Vault 的使用和生命周期以及各种已启用软删除的 Key Vault 对象。
- [如何将 Key Vault 软删除与 PowerShell 配合使用](soft-delete-powershell.md)介绍了 Key Vault 的使用和生命周期以及各种已启用软删除的 Key Vault 对象。

## <a name="integrated-with-key-vault"></a>与密钥保管库集成

这些文章介绍了使用 Key Vault 或与之集成的其他方案和服务。

- [Azure 磁盘加密](../../security/fundamentals/encryption-overview.md)利用 Windows 的行业标准 [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) 功能和 Linux 的 DM-Crypt 功能，为 OS 和数据磁盘提供卷加密。 该解决方案与 Azure 密钥保管库集成，可帮助你控制和管理密钥保管库订阅中的磁盘加密密钥和机密，同时确保虚拟机磁盘中的所有数据可在 Azure 存储中静态加密。

## <a name="key-vault-overviews-and-concepts"></a>Key Vault 概述和概念

- [Key Vault 软删除行为](overview-soft-delete.md)介绍了一种可以恢复已删除的对象的功能（不管是有意还是无意删除）。
- [Key Vault 客户端限制](overview-throttling.md)介绍了有关限制的基本概念，并针对应用提供了限制方法。

<!-- - [Key Vault security worlds](overview-security-worlds.md) describes the relationships between regions and security areas. -->

## <a name="social"></a>社交

- [密钥保管库博客](https://docs.microsoft.com/archive/blogs/kv/)
- [密钥保管库论坛](https://feedback.azure.com/forums/906355-azure-key-vault)

## <a name="supporting-libraries"></a>支持库

- [Azure Key Vault 核心库](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core)提供 IKey  和 IKeyResolver  接口，用于通过标识符查找密钥，以及使用密钥执行操作。
- [Azure 密钥保管库扩展](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions) 为 Azure 密钥保管库提供了扩展功能。
