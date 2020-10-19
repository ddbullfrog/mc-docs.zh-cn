---
title: include 文件
description: include 文件
services: app-service
author: kraigb
ms.service: app-service
ms.topic: include
origin.date: 09/24/2020
ms.date: 10/19/2020
ms.author: v-tawe
ms.custom: include file
ms.openlocfilehash: d20512040816a4a43cb28646edd3d7c1902938e9
ms.sourcegitcommit: e2e418a13c3139d09a6b18eca6ece3247e13a653
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/19/2020
ms.locfileid: "92170497"
---
# <a name="bash"></a>[Bash](#tab/bash)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
py -3 -m venv .venv
.venv\scripts\activate
pip install -r requirements.txt
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -3 -m venv .venv
.venv\scripts\activate
pip install -r requirements.txt
```

---   
