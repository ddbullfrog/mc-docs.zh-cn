---
title: current_principal_details() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 current_principal_details()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 11/08/2019
ms.date: 10/29/2020
ms.openlocfilehash: 22f6216750500109e0c830ac0079b5c64d6f7b76
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104209"
---
# <a name="current_principal_details"></a>current_principal_details()

返回运行查询的主体的详细信息。

## <a name="syntax"></a>语法

`current_principal_details()`

## <a name="returns"></a>返回

作为 `dynamic` 的当前主体的详细信息。

## <a name="example"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print d=current_principal_details()
```

|d|
|---|
|{<br>  "UserPrincipalName": "user@fabrikam.com",<br>  "IdentityProvider": "https://sts.windows.net",<br>  "Authority":"72f988bf-86f1-41af-91ab-2d7cd011db47",<br>  "Mfa":"True",<br>  "Type":"AadUser",<br>  "DisplayName":"James Smith (upn: user@fabrikam.com)",<br>  "ObjectId":"346e950e-4a62-42bf-96f5-4cf4eac3f11e",<br>  "FQN": null,<br>  "Notes": null<br>}|
