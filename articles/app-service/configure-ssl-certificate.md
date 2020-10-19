---
title: 添加和管理 TLS/SSL 证书
description: 在 Azure 应用服务中创建免费的证书、导入应用服务证书、导入 Key Vault 证书或购买应用服务证书。
tags: buy-ssl-certificates
ms.topic: tutorial
origin.date: 10/25/2019
ms.date: 10/19/2020
ms.author: v-tawe
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: 60c52586d1bfdcdc1a1994448f5eff67e79ad6e2
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170766"
---
# <a name="add-a-tlsssl-certificate-in-azure-app-service"></a>在 Azure 应用服务中添加 TLS/SSL 证书

[Azure 应用服务](overview.md)提供高度可缩放、自修复的 Web 托管服务。 本文介绍如何创建私有证书或公用证书，或将其上传或导入到应用服务中。 

将证书添加到应用服务应用或[函数应用](../azure-functions/index.yml)后，即可[使用它来保护自定义 DNS 名称](configure-ssl-bindings.md)或[在应用程序代码中使用它](configure-ssl-certificate-in-code.md)。

下表列出了用于在应用服务中添加证书的选项：

<!-- | Purchase an App Service certificate | A private certificate that's managed by Azure. It combines the simplicity of automated certificate management and the flexibility of renewal and export options. | -->

|选项|说明|
|-|-|
| 创建免费应用服务托管证书（预览版） | 如果只需保护 `www` [自定义域](app-service-web-tutorial-custom-domain.md)或应用服务中的任何非裸域，则可以轻松使用私有证书。 |
| 导入来自 Key Vault 的证书 | 这在使用 [Azure Key Vault](../key-vault/index.yml) 管理 [PKCS12 证书](https://wikipedia.org/wiki/PKCS_12)时很有用。 请参阅[私有证书要求](#private-certificate-requirements)。 |
| 上传私有证书 | 如果你已有第三方提供商提供的私有证书，则可以上传它。 请参阅[私有证书要求](#private-certificate-requirements)。 |
| 上传公用证书 | 公用证书不用于保护自定义域，但可以将其加载到代码中（如果需要它们来访问远程资源）。 |

## <a name="prerequisites"></a>先决条件

按照本操作方法指南操作：

- [创建应用服务应用](./index.yml)。
- 仅限免费证书：使用 [CNAME 记录](app-service-web-tutorial-custom-domain.md#map-a-cname-record)将子域（例如 `www.contoso.com`）映射到应用服务。

## <a name="private-certificate-requirements"></a>私有证书要求

> [!NOTE]
> Azure Web 应用**不**支持 AES256，并且所有 pfx 文件都应使用 TripleDES 进行加密。

<!-- The [free App Service Managed Certificate](#create-a-free-certificate-preview) or the [App Service certificate](#import-an-app-service-certificate) already satisfy the requirements of App Service.  -->

* 已导出为受密码保护的 PFX 文件
* 包含长度至少为 2048 位的私钥
* 包含证书链中的所有中间证书

若要保护 TLS 绑定中的自定义域，证书还有其他要求：

* 包含用于服务器身份验证的扩展密钥用法 (OID = 1.3.6.1.5.5.7.3.1)
* 已由受信任的证书颁发机构签名

> [!NOTE]
> 椭圆曲线加密 (ECC) 证书可用于应用服务，但本文不予讨论。 请咨询证书颁发机构，了解有关创建 ECC 证书的确切步骤。

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

<!-- no "create app service managed certificate" in private key certificates(.pfx) -->
<!-- 
## Create a free certificate (Preview)

The free App Service Managed Certificate is a turn-key solution for securing your custom DNS name in App Service. It's a fully functional TLS/SSL certificate that's managed by App Service and renewed automatically. The free certificate comes with the following limitations:

- Does not support wildcard certificates.
- Does not support naked domains.
- Is not exportable.
- Is not supported on App Service Environment (ASE)
- Does not support A records. For example, automatic renewal doesn't work with A records.

> [!NOTE]
> The free certificate is issued by DigiCert. For some top-level domains, you must explicitly allow DigiCert as a certificate issuer by creating a CAA domain record with the value: `0 issue digicert.com`.
> 

To create a free App Service Managed Certificate:

In the <a href="https://portal.azure.cn" target="_blank">Azure portal</a>, from the left menu, select **App Services** > **\<app-name>**.

From the left navigation of your app, select **TLS/SSL settings** > **Private Key Certificates (.pfx)** > **Create App Service Managed Certificate**.

![Create free certificate in App Service](./media/configure-ssl-certificate/create-free-cert.png)

Any non-naked domain that's properly mapped to your app with a CNAME record is listed in the dialog. Select the custom domain to create a free certificate for and select **Create**. You can create only one certificate for each supported custom domain.

When the operation completes, you see the certificate in the **Private Key Certificates** list.

![Create free certificate finished](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> To secure a custom domain with this certificate, you still need to create a certificate binding. Follow the steps in [Create binding](configure-ssl-bindings.md#create-binding).
>
-->

## <a name="import-an-app-service-certificate"></a>导入应用服务证书

<!--
If you purchase an App Service Certificate from Azure, Azure manages the following tasks:

- Takes care of the purchase process from GoDaddy.
- Performs domain verification of the certificate.
- Maintains the certificate in [Azure Key Vault](../key-vault/general/overview.md).
- Manages certificate renewal (see [Renew certificate](#renew-certificate)).
- Synchronize the certificate automatically with the imported copies in App Service apps.

To purchase an App Service certificate, go to [Start certificate order](#start-certificate-order).
-->

如果你已有一个有效的应用服务证书，则可以[将证书导入到应用服务中](#import-certificate-into-app-service)。

<!-- - [Manage the certificate](#manage-app-service-certificates), such as renew, rekey, and export it. -->

<!--
### Start certificate order

Start an App Service certificate order in the <a href="https://portal.azure.cn/#create/Microsoft.SSL" target="_blank">App Service Certificate create page</a>.

![Start App Service certificate purchase](./media/configure-ssl-certificate/purchase-app-service-cert.png)

Use the following table to help you configure the certificate. When finished, click **Create**.

| Setting | Description |
|-|-|
| Name | A friendly name for your App Service certificate. |
| Naked Domain Host Name | Specify the root domain here. The issued certificate secures *both* the root domain and the `www` subdomain. In the issued certificate, the Common Name field contains the root domain, and the Subject Alternative Name field contains the `www` domain. To secure any subdomain only, specify the fully qualified domain name of the subdomain here (for example, `mysubdomain.contoso.com`).|
| Subscription | The subscription that will contain the certificate. |
| Resource group | The resource group that will contain the certificate. You can use a new resource group or select the same resource group as your App Service app, for example. |
| Certificate SKU | Determines the type of certificate to create, whether a standard certificate or a wildcard certificate. |
| Legal Terms | Click to confirm that you agree with the legal terms. The certificates are obtained from GoDaddy. |

### Store in Azure Key Vault

Once the certificate purchase process is complete, there are few more steps you need to complete before you can start using this certificate. 

Select the certificate in the [App Service Certificates](https://portal.azure.cn/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) page, then click **Certificate Configuration** > **Step 1: Store**.

![Configure Key Vault storage of App Service certificate](./media/configure-ssl-certificate/configure-key-vault.png)

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) is an Azure service that helps safeguard cryptographic keys and secrets used by cloud applications and services. It's the storage of choice for App Service certificates.

In the **Key Vault Status** page, click **Key Vault Repository** to create a new vault or choose an existing vault. If you choose to create a new vault, use the following table to help you configure the vault and click Create. Create the new Key Vault inside the same subscription and resource group as your App Service app.

| Setting | Description |
|-|-|
| Name | A unique name that consists for alphanumeric characters and dashes. |
| Resource group | As a recommendation, select the same resource group as your App Service certificate. |
| Location | Select the same location as your App Service app. |
| Pricing tier | For information, see [Azure Key Vault pricing details](https://azure.microsoft.com/pricing/details/key-vault/). |
| Access policies| Defines the applications and the allowed access to the vault resources. You can configure it later, following the steps at [Grant several applications access to a key vault](../key-vault/key-vault-group-permissions-for-apps.md). |
| Virtual Network Access | Restrict vault access to certain Azure virtual networks. You can configure it later, following the steps at [Configure Azure Key Vault Firewalls and Virtual Networks](../key-vault/key-vault-network-security.md) |

Once you've selected the vault, close the **Key Vault Repository** page. The **Step 1: Store** option should show a green check mark for success. Keep the page open for the next step.

### Verify domain ownership

From the same **Certificate Configuration** page you used in the last step, click **Step 2: Verify**.

![Verify domain for App Service certificate](./media/configure-ssl-certificate/verify-domain.png)

Select **App Service Verification**. Since you already mapped the domain to your web app (see [Prerequisites](#prerequisites)), it's already verified. Just click **Verify** to finish this step. Click the **Refresh** button until the message **Certificate is Domain Verified** appears.

> [!NOTE]
> Four types of domain verification methods are supported: 
> 
> - **App Service** - The most convenient option when the domain is already mapped to an App Service app in the same subscription. It takes advantage of the fact that the App Service app has already verified the domain ownership.
> - **Domain** - Verify an [App Service domain that you purchased from Azure](manage-custom-dns-buy-domain.md). Azure automatically adds the verification TXT record for you and completes the process.
> - **Mail** - Verify the domain by sending an email to the domain administrator. Instructions are provided when you select the option.
> - **Manual** - Verify the domain using either an HTML page (**Standard** certificate only) or a DNS TXT record. Instructions are provided when you select the option.
-->

### <a name="import-certificate-into-app-service"></a>将证书导入到应用服务中

在 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>的左侧菜单中，选择“应用程序服务” > “\<app-name>” 。

在应用的左侧导航窗格中，选择“TLS/SSL 设置” > “私钥证书(.pfx)” > “导入应用服务证书”  。

![将应用服务证书导入到应用服务中](./media/configure-ssl-certificate/import-app-service-cert.png)

选择刚刚购买的证书，然后选择“确定”。

操作完成后，会在“私钥证书”列表中看到该证书。

![导入应用服务证书已完成](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> 若要使用此证书保护自定义域，仍需要创建证书绑定。 按照[创建绑定](configure-ssl-bindings.md#create-binding)中的步骤操作。
>

## <a name="import-a-certificate-from-key-vault"></a>导入来自 Key Vault 的证书

如果使用 Azure Key Vault 管理证书，则可以将 PKCS12 证书从 Key Vault 导入到应用服务中，前提是该证书[满足要求](#private-certificate-requirements)。

在 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>的左侧菜单中，选择“应用程序服务” > “\<app-name>” 。

在应用的左侧导航窗格中，选择“TLS/SSL 设置” > “私钥证书(.pfx)” > “导入 Key Vault 证书”  。

![将 Key Vault 证书导入到应用服务中](./media/configure-ssl-certificate/import-key-vault-cert.png)

使用下表来帮助选择证书。

| 设置 | 说明 |
|-|-|
| 订阅 | Key Vault 所属的订阅。 |
| Key Vault | 包含要导入的证书的保管库。 |
| 证书 | 从保管库中的 PKCS12 证书列表中进行选择。 保管库中的所有 PKCS12 证书都已通过其指纹列出，但在应用服务中并非支持所有证书。 |

操作完成后，会在“私钥证书”列表中看到该证书。 如果导入失败并出现错误，则证书不满足[应用服务的要求](#private-certificate-requirements)。

![导入 Key Vault 证书已完成](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> 若要使用此证书保护自定义域，仍需要创建证书绑定。 按照[创建绑定](configure-ssl-bindings.md#create-binding)中的步骤操作。
>

## <a name="upload-a-private-certificate"></a>上传私有证书

从证书提供者处获得证书以后，请执行此部分的步骤，使证书可供应用服务使用。

### <a name="merge-intermediate-certificates"></a>合并中间证书

如果证书颁发机构在证书链中提供了多个证书，则需按顺序合并证书。

若要执行此操作，请在文本编辑器中打开收到的所有证书。

创建名为 mergedcertificate.crt 的合并证书文件。 在文本编辑器中，将每个证书的内容复制到此文件。 证书的顺序应遵循证书链中的顺序，以你的证书开头，以根证书结尾， 如以下示例所示：

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

### <a name="export-certificate-to-pfx"></a>将证书导出为 PFX

使用在生成证书请求时所用的私钥导出合并的 TLS/SSL 证书。

如果使用 OpenSSL 生成证书请求，则已创建私钥文件。 若要将证书导出为 PFX，请运行以下命令。 将占位符 _&lt;private-key-file>_ 和 _&lt;merged-certificate-file>_ 分别替换为私钥和合并证书文件的路径。

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>  
```

出现提示时，定义导出密码。 稍后将 TLS/SSL 证书上传到应用服务时要使用此密码。

如果使用 IIS 或 Certreq.exe 生成证书请求，请将证书安装到本地计算机，然后[将证书导出为 PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx)。

### <a name="upload-certificate-to-app-service"></a>将证书上传到应用服务

现在可以将证书上传到应用服务了。

在 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>的左侧菜单中，选择“应用程序服务” > “\<app-name>” 。

在应用的左侧导航窗格中，选择“TLS/SSL 设置” > “私钥证书(.pfx)” > “上载证书”  。

![将私有证书上传到应用服务中](./media/configure-ssl-certificate/upload-private-cert.png)

在“PFX 证书文件”中选择 PFX 文件。 在“证书密码”中，键入导出 PFX 文件时创建的密码。 完成后，单击“上传”。 

操作完成后，会在“私钥证书”列表中看到该证书。

![上传证书文件已完成](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> 若要使用此证书保护自定义域，仍需要创建证书绑定。 按照[创建绑定](configure-ssl-bindings.md#create-binding)中的步骤操作。
>

## <a name="upload-a-public-certificate"></a>上传公用证书

支持使用 .cer 格式的公用证书。 

在 <a href="https://portal.azure.cn" target="_blank">Azure 门户</a>的左侧菜单中，选择“应用程序服务” > “\<app-name>” 。

在应用的左侧导航窗格中，单击“TLS/SSL 设置” > “公用证书(.cer)” > “上传公钥证书”  。

在“名称”中，键入证书的名称。 在“CER 证书文件”中，选择 CER 文件。

单击“上载” 。 

![将公用证书上传到应用服务中](./media/configure-ssl-certificate/upload-public-cert.png)

上传证书以后，请复制证书指纹并参阅[使证书可供访问](configure-ssl-certificate-in-code.md#make-the-certificate-accessible)。

<!-- ## Manage App Service certificates -->

## <a name="automate-with-scripts"></a>使用脚本自动执行

### <a name="azure-cli"></a>Azure CLI

```azurecli
#!/bin/bash

fqdn=<replace-with-www.{yourdomain}>
pfxPath=<replace-with-path-to-your-.PFX-file>
pfxPassword=<replace-with-your=.PFX-password>
resourceGroup=myResourceGroup
webappname=mywebapp$RANDOM

# Create a resource group.
az group create --location chinaeast2 --name $resourceGroup

# Create an App Service plan in Basic tier (minimum required by custom domains).
az appservice plan create --name $webappname --resource-group $resourceGroup --sku B1

# Create a web app.
az webapp create --name $webappname --resource-group $resourceGroup \
--plan $webappname

echo "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
read -p "Press [Enter] key when ready ..."

# Before continuing, go to your DNS configuration UI for your custom domain and follow the 
# instructions at https://aka.ms/appservicecustomdns to configure a CNAME record for the 
# hostname "www" and point it your web app's default domain name.

# Map your prepared custom domain name to the web app.
az webapp config hostname add --webapp-name $webappname --resource-group $resourceGroup \
--hostname $fqdn

# Upload the SSL certificate and get the thumbprint.
thumbprint=$(az webapp config ssl upload --certificate-file $pfxPath \
--certificate-password $pfxPassword --name $webappname --resource-group $resourceGroup \
--query thumbprint --output tsv)

# Binds the uploaded SSL certificate to the web app.
az webapp config ssl bind --certificate-thumbprint $thumbprint --ssl-type SNI \
--name $webappname --resource-group $resourceGroup

echo "You can now browse to https://$fqdn"
```

### <a name="powershell"></a>PowerShell

```azurepowershell
$fqdn="<Replace with your custom domain name>"
$pfxPath="<Replace with path to your .PFX file>"
$pfxPassword="<Replace with your .PFX password>"
$webappname="mywebapp$(Get-Random)"
$location="China East 2"

# Create a resource group.
New-AzResourceGroup -Name $webappname -Location $location

# Create an App Service plan in Free tier.
New-AzAppServicePlan -Name $webappname -Location $location `
-ResourceGroupName $webappname -Tier Free

# Create a web app.
New-AzWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
-ResourceGroupName $webappname

Write-Host "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
Read-Host "Press [Enter] key when ready ..."

# Before continuing, go to your DNS configuration UI for your custom domain and follow the 
# instructions at https://aka.ms/appservicecustomdns to configure a CNAME record for the 
# hostname "www" and point it your web app's default domain name.

# Upgrade App Service plan to Basic tier (minimum required by custom SSL certificates)
Set-AzAppServicePlan -Name $webappname -ResourceGroupName $webappname `
-Tier Basic

# Add a custom domain name to the web app. 
Set-AzWebApp -Name $webappname -ResourceGroupName $webappname `
-HostNames @($fqdn,"$webappname.chinacloudsites.cn")

# Upload and bind the SSL certificate to the web app.
New-AzWebAppSSLBinding -WebAppName $webappname -ResourceGroupName $webappname -Name $fqdn `
-CertificateFilePath $pfxPath -CertificatePassword $pfxPassword -SslState SniEnabled
```
## <a name="more-resources"></a>更多资源

* [在 Azure 应用服务中使用 TLS/SSL 绑定保护自定义 DNS 名称](configure-ssl-bindings.md)
* [实施 HTTPS](configure-ssl-bindings.md#enforce-https)
* [实施 TLS 1.1/1.2](configure-ssl-bindings.md#enforce-tls-versions)
* [在 Azure 应用服务中通过代码使用 TLS/SSL 证书](configure-ssl-certificate-in-code.md)
* [常见问题解答：应用服务证书](https://docs.azure.cn/app-service/faq-configuration-and-management/)
