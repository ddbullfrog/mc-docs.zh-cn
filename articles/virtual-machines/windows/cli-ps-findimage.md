---
title: 查找并使用 Azure 市场映像
description: 使用 Azure PowerSHell 来确定市场 VM 映像的发布者、产品/服务、SKU 和版本。
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
origin.date: 01/25/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: 598cb0c39038f88ba93552c43e22b3abb8a03816
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106364"
---
# <a name="find-and-use-vm-images-in-the-azure-marketplace-with-azure-powershell"></a>使用 Azure PowerShell 在 Azure 市场中查找并使用 VM 映像

本文介绍如何使用 Azure PowerShell 在 Azure 市场中查找 VM 映像。 然后，可以在创建 VM 时指定市场映像。

你还可以使用 [Azure 市场](https://market.azure.cn/marketplace/)店面、[Azure 门户](https://portal.azure.cn)或 [Azure CLI](../linux/cli-ps-findimage.md) 浏览可用的映像和产品/服务。 

[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]

## <a name="table-of-commonly-used-windows-images"></a>常用 Windows 映像表

此表显示了指示的发布者和产品/服务可用的 SKU 的子集。

| 发布者 | 产品/服务 | SKU |
|:--- |:--- |:--- |
| MicrosoftWindowsServer |WindowsServer |2019-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2019-Datacenter-Core |
| MicrosoftWindowsServer |WindowsServer |2019-Datacenter-with-Containers |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter-Server-Core |
| MicrosoftWindowsServer |WindowsServer |2016-Datacenter-with-Containers |
| MicrosoftWindowsServer |WindowsServer |2012-R2-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2012-Datacenter |
| MicrosoftSharePoint |MicrosoftSharePointServer |sp2019 |
| MicrosoftSQLServer |SQL2019-WS2016 |Enterprise |
| MicrosoftRServer |RServer-WS2016 |Enterprise |

## <a name="navigate-the-images"></a>浏览映像

在某个位置查找映像的一种方法是按顺序运行 [Get-AzVMImagePublisher](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimagepublisher)、[Get-AzVMImageOffer](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimageoffer) 和 [Get-AzVMImageSku](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimagesku) cmdlet：

1. 列出映像发布者。
2. 对于给定的发布者，列出其产品。
3. 对于给定的产品，列出其 SKU。

然后，对所选 SKU 运行 [Get-AzVMImage](https://docs.microsoft.com/powershell/module/az.compute/get-azvmimage)，列出要部署的版本。

1. 列出发布者：

    ```powershell
    Connect-AzAccount -Environment AzureChinaCloud
    $locName="<Azure location, such as China North>"
    Get-AzVMImagePublisher -Location $locName | Select PublisherName
    ```

2. 填写你选择的发布者名称并列出产品/服务：

    ```powershell
    $pubName="<publisher>"
    Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
    ```

3. 填写你选择的产品/服务名称并列出 SKU：

    ```powershell
    $offerName="<offer>"
    Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
    ```

4. 填写你选择的 SKU 名称并获取映像版本：

    ```powershell
    $skuName="<SKU>"
    Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
    ```

从 `Get-AzVMImage` 命令的输出中，可以选择要部署新虚拟机的版本映像。

以下示例显示了命令及其输出的完整序列：

```powershell
$locName="China North"
Get-AzVMImagePublisher -Location $locName | Select PublisherName
```

部分输出：

<!--MOONCCAKE: CUSTOMIZE-->

```
PublisherName
-------------
...
360-cn
A10Networks
a10networks-cn
afajr-cn
aibaby
aigauss
airdoc
airdoc-cn
alauda
AllMobilize
alteonva
Antshares
arcblock-cn
arctron
array_networks
array_networks-cn
AsiaInfo.DeepSecurity
attittu-cn
AzureChinaMarketplace
AzureDatabricks
...
```

<!--MOONCCAKE: CUSTOMIZE-->

对于“MicrosoftWindowsServer”发布者：

```powershell
$pubName="MicrosoftWindowsServer"
Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
```

输出：

<!--MOONCAKE CUSTOMIZE: invalid on Windows-HUB-->

```
Offer
-----
WindowsServer
WindowsServerSemiAnnual
```

<!--MOONCAKE CUSTOMIZE: invalid on Windows-HUB-->

对于“WindowsServer”产品/服务：

```powershell
$offerName="WindowsServer"
Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
```

部分输出：

<!--MOONCAKE CUSTOMIZE-->

```
Skus
----
2008-R2-SP1
2008-R2-SP1-smalldisk
2008-R2-SP1-zhcn
2012-Datacenter
2012-datacenter-gensecond
2012-Datacenter-smalldisk
2012-datacenter-smalldisk-g2
2012-Datacenter-zhcn
2012-datacenter-zhcn-g2
2012-R2-Datacenter
2012-r2-datacenter-gensecond
2012-R2-Datacenter-smalldisk
2012-r2-datacenter-smalldisk-g2
2012-R2-Datacenter-zhcn
2012-r2-datacenter-zhcn-g2
2016-Datacenter
2016-datacenter-gensecond
2016-datacenter-gs
2016-Datacenter-Server-Core
2016-datacenter-server-core-g2
2016-Datacenter-Server-Core-smalldisk
2016-datacenter-server-core-smalldisk-g2
2016-Datacenter-smalldisk
2016-datacenter-smalldisk-g2
2016-Datacenter-with-Containers
2016-datacenter-with-containers-g2
2016-datacenter-with-containers-gs
2016-Datacenter-zhcn
2016-datacenter-zhcn-g2
2016-Nano-Server
2019-Datacenter
2019-Datacenter-Core
2019-datacenter-core-g2
2019-Datacenter-Core-smalldisk
2019-datacenter-core-smalldisk-g2
2019-Datacenter-Core-with-Containers
...
```

然后，对于 *2019-Datacenter* SKU：

<!--MOONCAKE CUSTOMIZE-->

```powershell
$skuName="2019-Datacenter"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
```

现在可以将所选发布者、产品/服务、SKU 和版本合并到 URN 中（由“:”分隔的值）。 使用 [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) cmdlet 创建 VM 时，使用 `--image` 参数传递此 URN。 还可以将 URN 中的版本号替换为 "latest" 以获取映像的最新版本。

如果使用资源管理器模板部署 VM，请在 `imageReference` 属性中单独设置映像参数。

<!--Not Available on [template reference](https://docs.microsoft.com/azure/templates/microsoft.compute/virtualmachines)-->

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]

### <a name="view-plan-properties"></a>查看计划属性

若要查看映像的购买计划信息，请运行 `Get-AzVMImage` cmdlet。 如果输出中的 `PurchasePlan` 属性不是 `null`，则映像有条款，在以编程方式部署前需要接受该条款。  

<!--MOONCAKE CUSTOMIZE-->

例如， *Windows Server 2019 Datacenter* 映像没有附加条款，因此，`PurchasePlan` 信息为 `null`：

```powershell
$version = "2019.0.20190115"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName -Version $version
```

输出：

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/chinanorth/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2019.0.20190115
Location         : chinanorth
PublisherName    : MicrosoftWindowsServer
Offer            : WindowsServer
Skus             : 2019-Datacenter
Version          : 2019.0.20190115
FilterExpression :
Name             : 2019.0.20190115
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : null
DataDiskImages   : []

```

<!--MOONCAKE CUSTOMZIE: invalid *Data Science Virtual Machine - Windows 2016*-->

<!--Not Available on ### Accept the terms-->
<!--Not Available on ### Deploy using purchase plan parameters-->

## <a name="next-steps"></a>后续步骤

若要使用基本映像信息通过 `New-AzVM` cmdlet 快速创建虚拟机，请参阅[使用 PowerShell 创建 Windows 虚拟机](quick-create-powershell.md)。

有关使用 Azure 市场映像在共享映像库中创建自定义映像的更多信息，请参阅[在创建映像时提供 Azure 市场购买计划信息](../marketplace-images.md)。

<!-- Update_Description: update meta properties, wording update, update link -->