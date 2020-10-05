---
title: 创建 Azure 文件共享
titleSuffix: Azure Files
description: 如何使用 Azure 门户、PowerShell 或 Azure CLI 创建 Azure 文件共享。
author: WenJason
ms.service: storage
ms.topic: how-to
origin.date: 2/22/2020
ms.date: 09/28/2020
ms.author: v-jay
ms.subservice: files
ms.custom: devx-track-azurecli, references_regions
ms.openlocfilehash: cfd67082f3675e476696c4511c8a491e5f57f316
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026526"
---
# <a name="create-an-azure-file-share"></a>创建 Azure 文件共享
若要创建 Azure 文件共享，需要回答有关你将如何使用它的三个问题：

- **Azure 文件共享的性能要求是什么？**  
    Azure 文件存储提供在基于硬盘（基于 HDD）的硬件上托管的标准文件共享，以及在基于固态磁盘（基于 SSD）的硬件上托管的高级文件共享。

- **需要哪种大小的文件共享？**  
    标准文件共享最多可以涵盖 100 TiB 的空间，但默认不会启用此功能；如果所需的文件共享大于 5 TiB，则需要为存储帐户启用大型文件共享功能。 高级文件共享最多可扩展到 100 TiB 而无需任何特殊设置，但高级文件共享是经过预配的，而不像标准文件共享一样即付即用。 这意味着，预配一个远超所需大小的文件共享会增加存储的总成本。

- **Azure 文件共享在冗余方面的要求是什么？**  
    标准文件共享提供本地冗余 (LRS) 或异地冗余 (GRS) 存储，但是，只有本地冗余文件共享才支持大型文件共享功能。 高级文件共享不支持异地冗余。

有关这三个选项的详细信息，请参阅[规划 Azure 文件存储部署](storage-files-planning.md)。

## <a name="prerequisites"></a>先决条件
- 本文假设已创建一个 Azure 订阅。 如果还没有订阅，则在开始前创建一个 [1 元人民币的试用帐户](https://wd.azure.cn/zh-cn/pricing/1rmb-trial-full)。
- 如果你打算使用 Azure PowerShell，请[安装最新版本](https://docs.microsoft.com/powershell/azure/install-az-ps)。
- 如果你打算使用 Azure CLI，请[安装最新版本](/cli/install-azure-cli?view=azure-cli-latest)。

## <a name="create-a-storage-account"></a>创建存储帐户
Azure 文件共享将部署到存储帐户。存储帐户是代表存储共享池的顶级对象。 此存储池可用于部署多个文件共享。 

对于客户的不同存储方案，Azure 支持多种类型的存储帐户，但对于 Azure 文件存储，有两个主要类型的存储帐户。 需要创建的存储帐户类型取决于你是要创建标准文件共享还是要创建高级文件共享： 

- **常规用途版本 2 (GPv2) 存储帐户**：使用 GPv2 存储帐户可以在标准的/基于硬盘（基于 HDD）的硬件上部署 Azure 文件共享。 除了存储 Azure 文件共享以外，GPv2 存储帐户还可以存储其他存储资源，例如 Blob 容器、队列或表。 

- **FileStorage 存储帐户**：使用 FileStorage 存储帐户可以在高级/基于固态磁盘（基于 SSD）的硬件上部署 Azure 文件共享。 FileStorage 帐户只能用于存储 Azure 文件共享；其他存储资源（Blob 容器、队列、表等）都不能部署在 FileStorage 帐户中。

# <a name="portal"></a>[Portal](#tab/azure-portal)
若要通过 Azure 门户创建存储帐户，请在仪表板中选择“+ 创建资源”。 在出现的 Azure 市场搜索窗口中搜索“存储帐户”，并选择生成的搜索结果。 此时会显示存储帐户的概述页；选择“创建”继续执行存储帐户创建向导中的步骤。

![浏览器中存储帐户快速创建选项的屏幕截图](media/storage-how-to-create-file-share/create-storage-account-0.png)

#### <a name="the-basics-section"></a>“基本信息”部分
创建存储帐户所要完成的第一部分是标有“基本信息”的部分。 此部分包含创建存储帐户所需填写的所有字段。 若要创建 GPv2 存储帐户，请确保将“性能”单选按钮设置为“标准”，并在“帐户类型”下拉列表中选择“StorageV2 (常规用途 v2)”。

![一个屏幕截图，其中“性能”单选按钮已选择为“标准”，“帐户类型”已选择为“StorageV2”](media/storage-how-to-create-file-share/create-storage-account-1.png)

若要创建 FileStorage 存储帐户，请确保将“性能”单选按钮设置为“高级”，并在“帐户类型”下拉列表中选择“FileStorage”。

![一个屏幕截图，其中“性能”单选按钮已选择为“高级”，“帐户类型”已选择为“FileStorage”](media/storage-how-to-create-file-share/create-storage-account-2.png)

其他基本信息字段与存储帐户的选择无关：
- **订阅**：要部署到的存储帐户所在的订阅。 
- **资源组**：要部署到的存储帐户所在的资源组。 可以创建新的资源组，也可以使用现有资源组。 资源组是对 Azure 资源进行分组的逻辑容器。 在创建存储帐户时，可以选择创建新的资源组，也可以使用现有资源组。
- **存储帐户名称**：要创建的存储帐户资源的名称。 此名称必须全局唯一，但可以是所需的任何名称。 通过 SMB 装载 Azure 文件共享时，存储帐户名称将用作服务器名称。
- **位置**：要部署到的存储帐户所在的区域。 此区域可以是与资源组关联的区域，也可以是任何其他可用区域。
- **复制**：尽管此字段的标签为“复制”，但它实际上指的是“冗余”；这是所需的冗余级别：本地冗余 (LRS) 和异地冗余 (GRS)。 此下拉列表还包含读取访问权限异地冗余存储 (RA-GRS)，它不适用于 Azure 文件共享；在选择这些选项的情况下在存储帐户中创建的任何文件共享实际上是异地冗余的。 根据所在的区域或所选的存储帐户类型，可能不允许使用某些冗余选项。
- **访问层**：此字段不适用于 Azure 文件存储，因此可以选中任一单选按钮。

#### <a name="the-networking-blade"></a>“网络”边栏选项卡
在“网络”部分可以配置网络选项。 创建存储帐户时，这些设置是可选的，以后可根据需要对其进行配置。 有关这些选项的详细信息，请参阅 [Azure 文件存储的网络注意事项](storage-files-networking-overview.md)。

#### <a name="the-advanced-blade"></a>“高级”边栏选项卡
“高级”部分包含 Azure 文件共享的几项重要设置：

- **需要安全传输**：此字段指示存储帐户是否需要对与存储帐户的通信进行传输中加密。 建议将此功能保持启用状态，但是，如果需要 SMB 2.1 支持，则必须禁用此功能。 如果禁用加密，我们建议将存储帐户访问约束到包含服务终结点的虚拟网络。
- **大型文件共享**：此字段为最多涵盖 100 TiB 空间的文件共享启用存储帐户。 启用此功能会将存储帐户限制为只能使用本地冗余存储选项。 为大型文件共享启用 GPv2 存储帐户后，无法禁用大型文件共享功能。 FileStorage 存储帐户（高级文件共享的存储帐户）没有此选项，因为所有高级文件共享最多可以扩展到 100 TiB。

![适用于 Azure 文件存储的重要高级设置的屏幕截图](media/storage-how-to-create-file-share/create-storage-account-3.png)

“高级”选项卡中提供的其他设置（Blob 软删除、Azure Data Lake Storage Gen 2 的分层命名空间，以及 Blob 存储的 NFSv3）不适用于 Azure 文件存储。

#### <a name="tags"></a>Tags
标记是名称/值对，可让你通过将相同的标记应用到多个资源和资源组，对资源进行分类并查看合并的账单。 这些设置是可选的，可以在创建存储帐户后应用。

#### <a name="review--create"></a>查看 + 创建
创建存储帐户的最后一步是选中“查看 + 创建”选项卡上的“创建”按钮。 如果未填写存储帐户的所有必填字段，则此按钮将不可用。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
我们将在 PowerShell 中使用 `New-AzStorageAccount` cmdlet 创建存储帐户。 此 cmdlet 有许多选项；此处只显示了必需的选项。 若要详细了解高级选项，请参阅 [`New-AzStorageAccount` cmdlet 文档](https://docs.microsoft.com/powershell/module/az.storage/new-azstorageaccount)。

为了简化存储帐户和后续文件共享的创建，我们将在变量中存储多个参数。 可将变量内容替换为所需的任何值，但请注意，存储帐户名称必须全局唯一。

```azurepowershell
$resourceGroupName = "myResourceGroup"
$storageAccountName = "mystorageacct$(Get-Random)"
$region = "chinaeast2"
```

我们将使用以下命令创建能够存储标准 Azure 文件共享的存储帐户。 `-SkuName` 参数与所需的冗余类型相关；如果需要异地冗余存储帐户，则还必须删除 `-EnableLargeFileShare` 参数。

```azurepowershell
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Standard_LRS `
    -Location $region `
    -Kind StorageV2 `
    -EnableLargeFileShare
```

我们将使用以下命令创建能够存储高级 Azure 文件共享的存储帐户。 请注意，`-SkuName` 参数已更改为包含 `Premium` 和所需的冗余级别：本地冗余 (`LRS`)。 `-Kind` 参数是 `FileStorage` 而不是 `StorageV2`，因为必须在 FileStorage 存储帐户（而不是 GPv2 存储帐户）中创建高级文件共享。

```azurepowershell
$storAcct = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -SkuName Premium_LRS `
    -Location $region `
    -Kind FileStorage 
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
我们将在 Azure CLI 中使用 az storage account create 命令创建存储帐户。 此命令有许多选项；此处只显示了必需的选项。 若要详细了解高级选项，请参阅 [`az storage account create` 命令文档](/cli/storage/account)。

为了简化存储帐户和后续文件共享的创建，我们将在变量中存储多个参数。 可将变量内容替换为所需的任何值，但请注意，存储帐户名称必须全局唯一。

```azurecli
resourceGroupName="myResourceGroup"
storageAccountName="mystorageacct$RANDOM"
region="chinaeast2"
```

我们将使用以下命令创建能够存储标准 Azure 文件共享的存储帐户。 `--sku` 参数与所需的冗余类型相关；如果需要异地冗余存储帐户，则还必须删除 `--enable-large-file-share` 参数。

```azurecli
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind StorageV2 \
    --sku Standard_LRS \
    --enable-large-file-share \
    --output none
```

我们将使用以下命令创建能够存储高级 Azure 文件共享的存储帐户。 请注意，`--sku` 参数已更改为包含 `Premium` 和所需的冗余级别：本地冗余 (`LRS`)。 `--kind` 参数是 `FileStorage` 而不是 `StorageV2`，因为必须在 FileStorage 存储帐户（而不是 GPv2 存储帐户）中创建高级文件共享。

```azurecli
az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --kind FileStorage \
    --sku Premium_LRS \
    --output none
```

---

## <a name="create-file-share"></a>创建文件共享
创建存储帐户后，剩下的操作就是创建文件共享。 无论使用的是高级文件共享还是标准文件共享，此过程大部分都是相同的。 主要区别在于配额及其表示的内容。

标准文件共享是 Azure 文件共享的上限，最终用户不能超越此上限。 针对标准文件共享指定配额的主要目的是便于预算：“我不希望此文件共享增长到此临界点以外”。 如果未指定配额，则标准文件共享最多可以涵盖 100 TiB 的空间（如果未为存储帐户设置大型文件共享属性，则最多可以涵盖 5 TiB）。

对于高级文件共享，配额会重载以表示预配的大小。 预配的大小是将要对你计费的数量，与实际使用情况无关。 预配高级文件共享时，需要考虑两个因素：1) 需要从空间使用率角度考虑该共享的未来增长情况；2) 需要考虑工作负载所需的 IOPS。 每个预配的 GiB 都允许你使用额外的保留 IOPS 和突发 IOPS。 有关如何规划高级文件共享的详细信息，请参阅[预配高级文件共享](storage-files-planning.md#understanding-provisioning-for-premium-file-shares)。

# <a name="portal"></a>[Portal](#tab/azure-portal)
如果你刚刚创建了存储帐户，可以在部署屏幕中选择“转到资源”导航到该存储帐户。 如果以前已创建了存储帐户，可以通过它所在的资源组导航到该存储帐户。 进入存储帐户后，选择标有“文件共享”的磁贴（也可以通过存储帐户的目录导航到“文件共享”）。 

![“文件共享”磁贴的屏幕截图](media/storage-how-to-create-file-share/create-file-share-1.png)

在文件共享列表中，应会看到以前在此存储帐户中创建的任何文件共享；如果尚未创建任何文件共享，则会显示一个空表。 选择“+ 文件共享”创建新的文件共享。

此时屏幕上应会显示“新建文件共享”边栏选项卡。 填写“新建文件共享”边栏选项卡中的字段以创建文件共享：

- **名称**：要创建的文件共享的名称。
- **配额**：标准文件共享的文件共享配额；高级文件共享的文件共享预配大小。

选择“创建”以完成新共享的创建。 请注意，如果存储帐户位于虚拟网络中，那么除非客户端也在虚拟网络中，否则无法成功创建 Azure 文件共享。 也可以使用 Azure PowerShell `New-AzRmStorageShare` cmdlet 解决此时间点限制。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
可以使用 [`New-AzRmStorageShare`](https://docs.microsoft.com/powershell/module/az.storage/New-AzRmStorageShare) cmdlet 创建 Azure 文件共享。 以下 PowerShell 命令假设已根据前面有关使用 Azure PowerShell 创建存储帐户的部分中的定义，设置了变量 `$resourceGroupName` 和 `$storageAccountName`。 

> [!Important]  
> 对于高级文件共享，`-QuotaGiB` 参数指的是文件共享的预配大小。 文件共享的预配大小是将要对你计费的数量，与使用情况无关。 标准文件共享按使用情况计费，而不是按预配大小计费。

```azurepowershell
$shareName = "myshare"

New-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -QuotaGiB 1024 | Out-Null
```

> [!Note]  
> 文件共享的名称必须是全部小写。 若要全面且详细地了解如何为文件共享和文件命名，请参阅 [命名和引用共享、目录、文件和元数据](https://msdn.microsoft.com/library/azure/dn167011.aspx)。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
在使用 Azure CLI 创建 Azure 文件共享之前，必须先获取一个用于为文件共享创建操作授权的存储帐户密钥。 可以使用 [`az storage account keys list`](/cli/storage/account/keys) 命令实现此目的：

```azurecli
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')
```

获取存储帐户密钥后，可以使用 [`az storage share create`](/cli/storage/share) 命令创建 Azure 文件共享。 

> [!Important]  
> 对于高级文件共享，`--quota` 参数指的是文件共享的预配大小。 文件共享的预配大小是将要对你计费的数量，与使用情况无关。 标准文件共享按使用情况计费，而不是按预配大小计费。

```azurecli
shareName="myshare"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --quota 1024 \
    --output none
```

如果存储帐户包含在虚拟网络中，而调用此命令的计算机不在该虚拟网络中，则此命令将会失败。 可以根据前面所述使用 Azure PowerShell `New-AzRmStorageShare` cmdlet 来解决此时间点限制，或者在位于该虚拟网络中的计算机上执行 Azure CLI（包括通过 VPN 连接执行）。

---

> [!Note]  
> 文件共享的名称必须是全部小写。 若要全面且详细地了解如何为文件共享和文件命名，请参阅 [命名和引用共享、目录、文件和元数据](https://msdn.microsoft.com/library/azure/dn167011.aspx)。

### <a name="create-a-hot-or-cool-file-share"></a>创建热文件共享或冷文件共享
**常规用途 v2 (GPv2) 存储帐户**可以包含经过事务优化的热文件共享或冷文件共享（或二者的混合）。 事务优化共享可在所有 Azure 区域中提供，但热文件共享和冷文件共享仅在[部分区域](storage-files-planning.md#storage-tiers)提供。 可使用 Azure PowerShell 预览模块或 Azure CLI 创建热或冷文件共享。 

# <a name="portal"></a>[门户](#tab/azure-portal)
Azure 门户尚不支持创建热文件共享和冷文件共享，也不支持将现有的事务优化的文件共享移动到热或冷文件共享。 请查看有关使用 PowerShell 或 Azure CLI 创建文件共享的说明。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```PowerShell
# Update the Azure storage module to use the preview version. You may need to close and 
# reopen PowerShell before running this command. If you are running PowerShell 5.1, ensure 
# the following:
# - Run the below cmdlets as an administrator.
# - Have PowerShellGet 2.2.3 or later. Uncomment the following line to check.
# Get-Module -ListAvailable -Name PowerShellGet
Remove-Module -Name Az.Storage -ErrorAction SilentlyContinue
Uninstall-Module -Name Az.Storage
Install-Module -Name Az.Storage -RequiredVersion "2.1.1-preview" -AllowClobber -AllowPrerelease 

# Assuming $resourceGroupName and $storageAccountName from earlier in this document have already
# been populated. The access tier parameter may be TransactionOptimized, Hot, or Cool for GPv2 
# storage accounts. Standard tiers are only available in standard storage accounts. 
$shareName = "myhotshare"

New-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -AccessTier Hot

# You can also change an existing share's tier.
Update-AzRmStorageShare `
    -ResourceGroupName $resourceGroupName `
    -StorageAccountName $storageAccountName `
    -Name $shareName `
    -AccessTier Cool
```

> [!Note]  
> 预览版 Az.Storage PowerShell 模块中提供了通过 PowerShell 设置和更改层级的功能。 这些 cmdlet 或其输出在正式发布的 Az.Storage PowerShell 模块中发布之前可能会发生更改，因此，创建脚本时请记住这一点。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
最新的 Azure CLI 更新中提供了在特定层中创建文件共享或将文件共享移动到特定层的功能。 更新 Azure CLI 的操作特定于你使用的操作系统/Linux 发行版。 有关如何在你的系统上更新 Azure CLI 的说明，请参阅[安装 Azure CLI](/cli/install-azure-cli)。

```bash
# Assuming $resourceGroupName and $storageAccountName from earlier in this document have already
# been populated. The access tier parameter may be TransactionOptimized, Hot, or Cool for GPv2
# storage accounts. Standard tiers are only available in standard storage accounts.
shareName="myhotshare"

az storage share-rm create \
    --resource-group $resourceGroupName \
    --storage-account $storageAccountName \
    --name $shareName \
    --access-tier "Hot"
```

> [!Note]  
> 最新的 Azure CLI 包中提供了使用 `--access-tier` 参数设置层级的功能（预览版）。 此命令或其输出在标记为正式发布之前可能会发生更改，因此，创建脚本时请记住这一点。

---

## <a name="next-steps"></a>后续步骤
- [规划 Azure 文件存储的部署](storage-files-planning.md)。 
- [网络概述](storage-files-networking-overview.md)。
- 在 [Windows](storage-how-to-use-files-windows.md)、[macOS](storage-how-to-use-files-mac.md) 与 [Linux](storage-how-to-use-files-linux.md) 上连接和装载文件共享。