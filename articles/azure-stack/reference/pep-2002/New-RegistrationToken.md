---
title: Azure Stack Hub 的 New-RegistrationToken 特权终结点
description: PowerShell Azure Stack 特权终结点的引用 - New-RegistrationToken
author: WenJason
ms.topic: reference
origin.date: 04/27/2020
ms.date: 08/30/2020
ms.author: v-jay
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: 87d22d9cf3ec9653a36fad9d8d89c87bc2216319
ms.sourcegitcommit: bc10b8dd34a2de4a38abc0db167664690987488d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91437642"
---
# <a name="new-registrationtoken"></a>New-RegistrationToken

## <a name="synopsis"></a>摘要
创建新的注册令牌。

## <a name="syntax"></a>语法

```
New-RegistrationToken [[-MsAssetTag] <Object>] [[-AgreementNumber] <Object>] [[-BillingModel] <Object>]
 [-MarketplaceSyndicationEnabled] [-AsJob]
```

## <a name="parameters"></a>parameters

### <a name="-billingmodel"></a>-BillingModel
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 1
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-marketplacesyndicationenabled"></a>-MarketplaceSyndicationEnabled
 

```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-agreementnumber"></a>-AgreementNumber
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 2
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-msassettag"></a>-MsAssetTag
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: 3
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-asjob"></a>-AsJob


```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="next-steps"></a>后续步骤

有关如何访问和使用特权终结点的信息，请参阅[使用 Azure Stack Hub 中的特权终结点](../../operator/azure-stack-privileged-endpoint.md)。
