---
title: Azure Analysis Services 服务器别名 | Azure
description: 了解如何创建 Azure Analysis Services 服务器名称别名。 然后，用户可以使用较短的别名，而不是服务器名称连接到服务器。
ms.service: azure-analysis-services
ms.topic: conceptual
origin.date: 06/16/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: 11/25/2019
ms.author: v-yeche
ms.reviewer: minewiskan
ms.openlocfilehash: 8197739a732f7d7f12baa483f620421348990c27
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92469958"
---
# <a name="alias-server-names"></a>服务器别名

有了服务器别名，用户就可以在连接到 Azure Analysis Services 服务器时使用较短的别名来代替服务器名称。 从客户端应用程序进行连接时，可以使用 **link://** 协议格式将别名指定为终结点。 然后，终结点会返回进行连接所需的真实的服务器名称。

服务器别名适用于：

- 在服务器间迁移模型而不影响用户。 
- 用户更容易记住友好的服务器名称。 
- 在一天的不同时间将用户定向到不同的服务器。 
- 将不同区域的用户定向到从地理上来说更近的实例，这与使用 Azure 流量管理器时的情况类似。 

任何返回有效的 Azure Analysis Services 服务器名称的 HTTPS 终结点都可以充当别名。 终结点必须通过端口 443 支持 HTTPS，且不得在 URI 中指定端口。

:::image type="content" source="media/analysis-services-alias/aas-alias-browser.png" alt-text="使用链接格式的别名":::

从客户端进行连接时，服务器别名是使用 **link://** 协议格式输入的。 例如，在 Power BI Desktop 中：

:::image type="content" source="media/analysis-services-alias/aas-alias-connect-pbid.png" alt-text="使用链接格式的别名":::

## <a name="create-an-alias"></a>创建别名

若要创建别名终结点，可以使用任何返回有效的 Azure Analysis Services 服务器名称的方法。 例如，可以引用 Azure Blob 存储中包含真实服务器名称的文件，也可以创建并发布 ASP.NET Web 窗体应用程序。

此示例的 ASP.NET Web 窗体应用程序在 Visual Studio 中创建。 从 Default.aspx 页面删除页引用和用户控制。 Default.aspx 的内容只是以下 Page 指令：

```
<%@ Page Title="Home Page" Language="C#" AutoEventWireup="true" CodeBehind="Default.aspx.cs" Inherits="FriendlyRedirect._Default" %>
```

Default.aspx.cs 中的 Page_Load 事件使用 Response.Write() 方法返回 Azure Analysis Services 服务器名称。

```
protected void Page_Load(object sender, EventArgs e)
{
    this.Response.Write("asazure://<region>.asazure.chinacloudapi.cn/<servername>");
}
```

## <a name="see-also"></a>另请参阅

[客户端库](https://docs.microsoft.com/analysis-services/client-libraries?view=azure-analysis-services-current)   
[从 Power BI Desktop 进行连接](analysis-services-connect-pbi.md)

<!-- Update_Description: update meta properties, wording update, update link -->