---
title: 将自定义 VM 映像添加到 Azure Stack Hub
description: 了解如何在 Azure Stack Hub 中添加或删除自定义 VM 映像。
author: WenJason
ms.topic: how-to
ms.service: azure-stack
origin.date: 09/08/2020
ms.date: 10/12/2020
ms.author: v-jay
ms.reviewer: kivenkat
ms.lastreviewed: 9/8/2020
ms.openlocfilehash: c339bb19903d80ef29ad4beb6d3bc5511d416575
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437744"
---
# <a name="add-and-remove-a-custom-vm-image-to-azure-stack-hub"></a>在 Azure Stack Hub 中添加和删除自定义 VM 映像

在 Azure Stack Hub 中，作为操作员，你可以将虚拟机 (VM) 自定义映像添加到市场供用户使用。 可以通过管理员门户或 Windows PowerShell 将 VM 映像添加到 Azure Stack Hub 市场。 使用 Azure 市场中的映像作为自定义映像的基础，或使用 Hyper-V 创建自己的映像。

## <a name="add-an-image"></a>添加映像

可以在用户指南的“计算”部分中找到有关添加通用映像和专用映像的说明。 在为用户提供通用映像之前，需要先创建该映像。 有关说明，请参阅[将 VM 移动到 Azure Stack Hub 概述](/azure-stack/user/vm-move-overview)。 创建可用于租户的映像时，请使用 Azure Stack Hub 管理门户或管理员终结点，而不是用户门户或租户目录终结点。

有两个选项可用于向用户提供映像：

- **提供只能通过 Azure 资源管理器访问的映像**  
  如果通过 Azure Stack Hub 管理门户中的“计算” > “映像”添加映像，则你的所有租户都可以访问该映像。 但是，你的用户需要使用 Azure 资源管理器模板来访问它。 它将在 Azure Stack Hub 市场中不可见。

- **通过 Azure Stack Hub 市场提供映像**  
    通过 Azure Stack Hub 管理门户添加映像后，便可以创建市场产品/服务了。 有关说明，请参阅[创建并发布自定义 Azure Stack Hub 市场项](azure-stack-create-and-publish-marketplace-item.md)。


## <a name="add-a-platform-image"></a>添加平台映像

若要将平台映像添加到 Azure Stack Hub，请使用 Azure Stack Hub 管理员门户或终结点，也可以使用 PowerShell。 需要已创建通用 VHD。 有关说明，请参阅[将 VM 移动到 Azure Stack Hub 概述](/azure-stack/user/vm-move-overview)。

### <a name="portal"></a>[Portal](#tab/image-add-portal)

以 Azure Stack Hub 操作员的身份使用门户添加 VM 映像。

1. 以操作员身份登录到 Azure Stack Hub。 在菜单中选择“所有服务” > “计算” > “映像” > “添加”。   

   ![添加 VM 映像](./media/azure-stack-add-vm-image/tca4.png)

2. 在“创建映像”下，输入“发布者”、“套餐”、“SKU”、“版本”和 OS 磁盘 blob URI。 然后选择“创建”，开始创建 VM 映像。

   ![自定义映像旁加载 UI](./media/azure-stack-add-vm-image/tca5.png)

   成功创建映像后，VM 映像状态会更改为“已成功”。

3. 添加映像时，它仅适用于基于 Azure 资源管理器的模板和 PowerShell 部署。 若要将映像作为市场项提供给用户，请使用[创建和发布市场项](azure-stack-create-and-publish-marketplace-item.md)一文中的步骤发布市场项 请务必记下“发布者”、“套餐”、“SKU”和“版本”的值。**** **** **** **** 在自定义 .azpkg 中编辑资源管理器模板和 Manifest.json 时，需要用到这些值。

### <a name="powershell"></a>[PowerShell](#tab/image-add-ps)

 以 Azure Stack Hub 操作员的身份使用 PowerShell 添加 VM 映像。

1. [安装适用于 Azure Stack Hub 的 PowerShell](azure-stack-powershell-install.md)。  

2. 以操作员身份登录到 Azure Stack Hub。 有关说明，请参阅[以操作员身份登录到 Azure Stack Hub](azure-stack-powershell-configure-admin.md)。

3. 使用权限提升的提示符打开 PowerShell，并运行：

   ```powershell
    Add-AzsPlatformimage -publisher "<publisher>" `
      -offer "<Offer>" `
      -sku "<SKU>" `
      -version "<#.#.#>" `
      -OSType "<OS type>" `
      -OSUri "<OS URI>"
   ```

   **Add-AzsPlatformimage** cmdlet 指定 Azure 资源管理器模板用来引用 VM 映像的值。 这些值包括：
   - **publisher**  
     例如： `Canonical`  
     VM 映像的**发布者**名称段，供用户在部署映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **offer**  
     例如： `UbuntuServer`  
     VM 映像的**套餐**名称段，供用户在部署 VM 映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **sku**  
     例如： `14.04.3-LTS`  
     VM 映像的 SKU 名称段，供用户在部署 VM 映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **version**  
     例如： `1.0.0`  
     VM 映像的版本，供用户在部署 VM 映像时使用。 此版本采用 **\#.\#.\#** 格式。 不要在此字段中包含空格或其他特殊字符。  
   - **osType**  
     例如： `Linux`  
     映像的 **osType** 必须为 **Windows** 或 **Linux**。  
   - **OSUri**  
     例如： `https://storageaccount.blob.core.chinacloudapi.cn/vhds/Ubuntu1404.vhd`  
     可以指定 `osDisk` 的 Blob 存储 URI。  

     有关详细信息，请参阅 [Add-AzsPlatformimage](https://docs.microsoft.com/powershell/module/azs.compute.admin/add-azsplatformimage) cmdlet 的 PowerShell 参考。

4. 添加映像时，它仅适用于基于 Azure 资源管理器的模板和 PowerShell 部署。 若要将映像作为市场项提供给用户，请使用[创建和发布市场项](azure-stack-create-and-publish-marketplace-item.md)一文中的步骤发布市场项 请务必记下“发布者”、“套餐”、“SKU”和“版本”的值。**** **** **** **** 在自定义 .azpkg 中编辑资源管理器模板和 Manifest.json 时，需要用到这些值。

---

## <a name="remove-a-platform-image"></a>删除平台映像

可以使用门户或 PowerShell 删除平台映像。

### <a name="portal"></a>[Portal](#tab/image-rem-portal)

若要以 Azure Stack Hub 操作员的身份使用 Azure Stack Hub 门户删除 VM 映像，请执行以下步骤：

1. 打开 Azure Stack Hub [管理员门户](https://portal.azure.cn/signin/index)。

2. 如果 VM 映像有关联的市场项，请选择“市场管理”，然后选择要删除的 VM 市场项。

3. 如果 VM 映像没有关联的市场项，请导航到“所有服务”>“计算”>“VM 映像”，然后选择 VM 映像旁边的省略号 ( **...** )。

4. 选择“删除” 。

### <a name="powershell"></a>[PowerShell](#tab/image-rem-ps)

若要以 Azure Stack Hub 操作员的身份使用 PowerShell 删除 VM 映像，请执行以下步骤：

1. [安装适用于 Azure Stack Hub 的 PowerShell](azure-stack-powershell-install.md)。

2. 以操作员身份登录到 Azure Stack Hub。

3. 使用权限提升的提示符打开 PowerShell，并运行：

   ```powershell  
   Remove-AzsPlatformImage `
    -publisher "<Publisher>" `
    -offer "<Offer>" `
    -sku "<SKU>" `
    -version "<Version>" `
   ```

   **Remove-AzsPlatformImage** cmdlet 指定 Azure 资源管理器模板用来引用 VM 映像的值。 这些值包括：
   - **publisher**  
     例如： `Canonical`  
     VM 映像的**发布者**名称段，供用户在部署映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **offer**  
     例如： `UbuntuServer`  
     VM 映像的**套餐**名称段，供用户在部署 VM 映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **sku**  
     例如： `14.04.3-LTS`  
     VM 映像的 SKU 名称段，供用户在部署 VM 映像时使用。 不要在此字段中包含空格或其他特殊字符。  
   - **version**  
     例如： `1.0.0`  
     VM 映像的版本，供用户在部署 VM 映像时使用。 此版本采用 **\#.\#.\#** 格式。 不要在此字段中包含空格或其他特殊字符。  

     有关 **Remove-AzsPlatformImage** cmdlet 的详细信息，请参阅 Azure PowerShell [Azure Stack Hub 操作员模块文档](https://docs.microsoft.com/powershell/azure/azure-stack/overview)。
---
## <a name="next-steps"></a>后续步骤

- [创建并发布自定义 Azure Stack Hub 市场项](azure-stack-create-and-publish-marketplace-item.md)
- [预配虚拟机](../user/azure-stack-create-vm-template.md)
