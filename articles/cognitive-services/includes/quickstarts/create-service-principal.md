---
title: 创建 Azure 服务主体
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: 604173e54990e634db08c39475e3cac34c25fee0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106477"
---
## <a name="create-an-azure-service-principal"></a>创建 Azure 服务主体

若要使应用程序与 Azure 帐户交互，需要使用 Azure 服务主体来管理权限。 请按照[创建 Azure 服务主体](https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-4.4.0&viewFallbackFrom=azps-3.3.0)中的说明操作。

创建服务主体时，你会看到它有一个机密值、一个 ID 和一个应用程序 ID。 将应用程序 ID 和机密保存到某个临时位置，以供后续步骤使用。

