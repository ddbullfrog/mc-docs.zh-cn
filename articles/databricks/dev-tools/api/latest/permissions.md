---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 权限 API - Azure Databricks
description: 权限 API 使 Azure Databricks 工作区管理员可以控制各种业务对象的权限。
ms.openlocfilehash: aa1a801a32c79577dacde742d558d85e1fbe4c3d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937671"
---
<!--IMPORTANT:  For docs.databricks.com, this page will get STOMPED ON by doc tools with the OpenAPI rendered by redoc. Title shows up in TOC.-->

# <a name="permissions-api"></a>权限 API

> [!IMPORTANT]
>
> 此功能目前以[公共预览版](../../../release-notes/release-types.md)提供。

权限 API 可让你管理以下项的权限：

* 令牌
* 群集
* 池
* 作业
* 笔记本
* 文件夹（目录）
* MLflow 注册模型

权限 API 作为 OpenAPI 3.0 规范提供，你可以下载它并在喜欢的 OpenAPI 编辑器中将它作为结构化 API 参考进行查看。

* [下载 OpenAPI 规范](../../../_static/api-refs/permissions-azure.yaml)
* [在 Redocly 中查看](https://redocly.github.io/redoc/?url=https://docs.microsoft.com/azure/databricks/_static/api-refs/permissions-azure.yaml)：此链接会立即将 OpenAPI 规范作为结构化 API 参考打开，便于用户查看。
* [在 Postman 中查看](https://learning.postman.com/)：Postman 是一个必须[下载](https://www.postman.com/downloads/)到计算机的应用。 下载完以后，可以文件或 URL 形式[导入 OpenAPI 规范](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/#importing-api-specifications)。
* [在 Swagger 编辑器中查看](https://editor.swagger.io/)：在联机 Swagger 编辑器中，转到“文件”菜单，然后单击“导入文件”以导入并查看下载的 OpenAPI 规范。

> [!IMPORTANT]
>
> 要访问 Databricks REST API，必须[进行身份验证](authentication.md)。