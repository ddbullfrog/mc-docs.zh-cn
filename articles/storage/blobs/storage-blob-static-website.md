---
title: Azure 存储中的静态网站托管
description: Azure 存储静态网站托管提供经济高效、可缩放的解决方案用于托管新式 Web 应用程序。
author: WenJason
ms.service: storage
ms.topic: how-to
ms.author: v-jay
ms.reviewer: dineshm
origin.date: 09/04/2020
ms.date: 09/28/2020
ms.subservice: blobs
ms.custom: devx-track-javascript
ms.openlocfilehash: a8a3b0c1676cd4e63214a9366bafce30f57b290b
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026627"
---
# <a name="static-website-hosting-in-azure-storage"></a>Azure 存储中的静态网站托管

可以直接通过名为 *$web* 的存储容器提供静态内容（HTML、CSS、JavaScript 和图像文件）。 通过将内容托管在 Azure 存储中，你可以使用无服务器体系结构，包括 [Azure Functions](/azure-functions/functions-overview) 和其他平台即服务 (PaaS) 服务。  如果你不需要 Web 服务器来呈现内容，Azure 存储静态网站托管是一个不错的选择。

如果需要 Web 服务器来呈现内容，可以使用 [Azure 应用服务](/app-service/)。

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

> [!NOTE]
> 确保创建常规用途 v2 标准存储帐户。 静态网站在任何其他类型的存储帐户中都不可用。

## <a name="setting-up-a-static-website"></a>设置静态网站

静态网站托管是必须在存储帐户上启用的一项功能。

若要启用静态网站托管，请选择默认文件的名称，然后可以提供自定义 404 页面的路径。 如果帐户中尚不存在名为 **$web** 的 blob 存储容器，则会为你创建一个。 将站点的文件添加到此容器。

有关分步指南，请参阅[在 Azure 存储中托管静态网站](storage-blob-static-website-how-to.md)。

![Azure 存储静态网站指标 - 指标](./media/storage-blob-static-website/storage-blob-static-website-blob-container.png)

**$web** 容器中的文件区分大小写，通过匿名访问请求来提供，只能在读取操作中使用。

## <a name="uploading-content"></a>上传内容

可以使用下列工具中的任何一种将内容上传到 **$web** 容器：

> [!div class="checklist"]
> * [Azure CLI](storage-blob-static-website-how-to.md?tabs=azure-cli)
> * [Azure PowerShell 模块](storage-blob-static-website-how-to.md?tabs=azure-powershell)
> * [AzCopy](../common/storage-use-azcopy-v10.md)
> * [Azure 存储资源管理器](https://azure.microsoft.com/features/storage-explorer/)
> * [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/)
> * [Visual Studio Code 扩展](https://docs.microsoft.com/azure/developer/javascript/tutorial-vscode-static-website-node-01)

## <a name="viewing-content"></a>查看内容

用户可以在浏览器中使用网站的公共 URL 来查看站点内容。 可以使用 Azure 门户、Azure CLI 或 PowerShell 查找 URL。 请参阅[查找网站 URL](storage-blob-static-website-how-to.md#portal-find-url)。

如果服务器返回 404 错误，并且你在启用网站时未指定错误文档，则会向用户返回默认 404 页面。

> [!NOTE]
> 静态网站不支持 [CORS](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services)。

### <a name="regional-codes"></a>区域代码

站点的 URL 包含区域代码。 例如，URL `https://contosoblobaccount.z4.web.core.chinacloudapi.cn/` 包含区域代码 `z4`。

尽管该代码必须保留在 URL 中，但它仅供内部使用，无需以任何其他方式使用该代码。

当用户打开站点并且未指定特定文件（例如 `https://contosoblobaccount.z4.web.core.chinacloudapi.cn`）时，将出现你在启用静态网站托管时指定的索引文档。

### <a name="secondary-endpoints"></a>辅助终结点

如果[在次要区域中设置冗余](../common/storage-redundancy.md#redundancy-in-a-secondary-region)，则还可以使用辅助终结点访问网站内容。 由于数据以异步方式复制到次要区域，因此辅助终结点上可用的文件并不总是与主终结点上可用的文件同步。

## <a name="impact-of-the-setting-the-public-access-level-of-the-web-container"></a>设置 Web 容器的公共访问级别的影响

你可以修改 **$web** 容器的公共访问级别，但这不会影响主静态网站终结点，因为这些文件是通过匿名访问请求来提供的。 这意味着对所有文件的公共（只读）访问权限。

以下屏幕截图显示了 Azure 门户中的公共访问级别设置：

![显示如何在门户中设置公共访问级别的屏幕截图](./media/anonymous-read-access-configure/configure-public-access-container.png)

尽管主静态网站终结点不受影响，但更改公共访问级别会影响主 blob 服务终结点。

例如，如果将 **$web** 容器的公共访问级别从**专用（不允许匿名访问）** 更改为 **Blob（仅允许匿名读取 blob）** ，则对主静态网站终结点 `https://contosoblobaccount.z4.web.core.chinacloudapi.cn/index.html` 的公共访问级别不会更改。

但是，对主 blob 服务终结点 `https://contosoblobaccount.blob.core.chinacloudapi.cn/$web/index.html` 的公共访问权限将从专用更改为公共。 现在，用户可以使用这两个终结点中的任意一个来打开该文件。

## <a name="mapping-a-custom-domain-to-a-static-website-url"></a>将自定义域映射到静态网站 URL

你可以使静态网站可通过自定义域进行访问。

为自定义域启用 HTTP 访问更为容易，因为 Azure 存储原本就支持它。

如果将存储帐户配置为通过 HTTPS [要求进行安全传输](../common/storage-require-secure-transfer.md)，则用户必须使用 HTTPS 终结点。

> [!TIP]
> 考虑在 Azure 上托管域。 有关详细信息，请参阅[在 Azure DNS 中托管域](../../dns/dns-delegate-domain-azure-dns.md)。

## <a name="pricing"></a>定价

可以免费启用静态网站托管。 只会针对你的站点利用的 blob 存储和运营成本进行计费。 如需详细了解 Azure Blob 存储价格，请参阅 [Azure Blob 存储定价页](https://azure.cn/pricing/details/storage/blobs/)。

## <a name="metrics"></a>指标

可以在静态网站页面上启用指标。 启用指标后，指标仪表板会报告有关 $web 容器中的文件的流量统计信息。

若要在静态网站页面上启用指标，请参阅[在静态网站页面上启用指标](storage-blob-static-website-how-to.md#metrics)。

## <a name="next-steps"></a>后续步骤

* [在 Azure 存储中托管静态网站](storage-blob-static-website-how-to.md)
* [将自定义域映射到 Azure Blob 存储终结点](storage-custom-domain-name.md)
* [Azure Functions](/azure-functions/functions-overview)
* [Azure 应用服务](/app-service/overview)
* [教程：在 Azure DNS 中托管域](../../dns/dns-delegate-domain-azure-dns.md)
