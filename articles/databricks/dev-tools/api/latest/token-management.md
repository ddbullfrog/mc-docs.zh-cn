---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/01/2020
title: 令牌管理 API - Azure Databricks
description: 令牌管理 API 为 Azure Databricks 帐户管理员提供了对其工作区中个人访问令牌的见解和控制。
ms.openlocfilehash: 1f05f583d1f75f3b4dd9e9b12f997fa9a565ce2a
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937820"
---
<!--IMPORTANT: THIS IS A STUB FILE ONLY. The docs.databricks.com version of this page will get STOMPED ON by doc tools with the OpenAPI rendered by redoc. Title shows up in TOC.-->

# <a name="token-management-api"></a>令牌管理 API

Azure Databricks 管理员可以使用令牌管理 API 管理其用户的 Azure Databricks 个人访问令牌。 作为管理员，你可以：

* 监视和撤销用户的个人访问令牌。
* 控制工作区中未来令牌的生存期。

还可以通过[权限 API](permissions.md) 或[管理员控制台](../../../administration-guide/access-control/tokens.md)来控制哪些用户可以创建和使用令牌。

令牌管理 API 作为 OpenAPI 3.0 规范提供，你可以下载它并在喜欢的 OpenAPI 编辑器中将它作为结构化 API 参考进行查看。

* [下载 OpenAPI 规范](../../../_static/api-refs/token-management-azure.yaml)
* [在 Redocly 中查看](https://redocly.github.io/redoc/?url=https://docs.microsoft.com/azure/databricks/_static/api-refs/token-management-azure.yaml)：此链接会立即将 OpenAPI 规范作为结构化 API 参考打开，便于用户查看。
* [在 Postman 中查看](https://learning.postman.com/)：Postman 是一个必须[下载](https://www.postman.com/downloads/)到计算机的应用。 下载完以后，可以文件或 URL 形式[导入 OpenAPI 规范](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/#importing-api-specifications)。
* [在 Swagger 编辑器中查看](https://editor.swagger.io/)：在联机 Swagger 编辑器中，转到“文件”菜单，然后单击“导入文件”以导入并查看下载的 OpenAPI 规范。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。