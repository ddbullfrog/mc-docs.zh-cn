---
ms.service: virtual-machines-sql
ms.topic: include
origin.date: 11/25/2018
author: rockboyfor
ms.date: 10/26/2020
ms.author: v-yeche
ms.openlocfilehash: bed17b05fa6977a62cc3987564dc140a410bfc15
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92499191"
---
## <a name="start-your-powershell-session"></a>启动 PowerShell 会话
 

运行 [**Connect-Az Account**](https://docs.microsoft.com/powershell/module/Az.Accounts/Connect-AzAccount) cmdlet，然后便会看到可输入凭据的登录屏幕。 使用与登录 Azure 门户相同的凭据。

```powershell
Connect-AzAccount -Environment AzureChinaCloud
```

如果有多个订阅，请使用 [**Set-AzContext**](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext) cmdlet 选择 PowerShell 会话应使用的订阅。 若要查看当前 PowerShell 会话正在使用哪个订阅，请运行 [**Get-AzContext**](https://docs.microsoft.com/powershell/module/az.accounts/get-azcontext)。 若要查看所有订阅，请运行 [**Get-AzSubscription**](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription)。

```powershell
Set-AzContext -SubscriptionId '4cac86b0-1e56-bbbb-aaaa-000000000000'
```

<!-- Update_Description: update meta properties, wording update, update link -->