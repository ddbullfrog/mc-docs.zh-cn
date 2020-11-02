---
title: 存储连接字符串 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的存储连接字符串。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/23/2020
ms.date: 10/29/2020
ms.openlocfilehash: 1e8a1e85c8bce05203530f6a01527599c2372873
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93105534"
---
# <a name="storage-connection-strings"></a>存储连接字符串

一些 Kusto 命令可指示 Kusto 与外部存储服务交互。 例如，可以指示 Kusto 将数据导出到 Azure 存储 Blob，在这种情况下，需要提供特定的参数（例如，存储帐户名称或 Blob 容器）。

Kusto 支持以下存储提供程序：


* Azure 存储 Blob 存储提供程序
* Azure Data Lake Storage 存储提供程序

每种类型的存储提供程序都定义了用于描述存储资源的连接字符串格式以及访问这些资源的方式。
Kusto 使用 URI 格式描述这些存储资源以及访问这些资源所需的属性（如安全凭据）。


|提供程序                   |Scheme    |URI 模板                          |
|---------------------------|----------|--------------------------------------|
|Azure 存储 Blob         |`https://`|`https://`*Account*`.blob.core.chinacloudapi.cn/`*Container* [`/`*BlobName* ][`?`*SasKey* \| `;`*AccountKey* ]|
|Azure Data Lake Store Gen 2|`abfss://`|`abfss://`*Filesystem*`@`*Account*`.dfs.core.chinacloudapi.cn/`*PathToDirectoryOrFile* [`;`*CallerCredentials* ]|
|Azure Data Lake Store Gen 1|`adl://`  |`adl://`*Account* .azuredatalakestore.net/ *PathToDirectoryOrFile* [`;`*CallerCredentials* ]|

## <a name="azure-storage-blob"></a>Azure 存储 Blob

此提供程序最常用，在所有方案中均受支持。
访问资源时，必须为提供程序提供凭据。 支持通过两种机制提供凭据：

* 使用 Azure 存储 Blob 的标准查询 (`?sig=...`) 提供共享访问签名 (SAS) 密钥。 当 Kusto 需要在有限的时间内访问资源时，请使用此方法。
* 提供存储帐户密钥 (`;ljkAkl...==`)。 当 Kusto 需要持续访问资源时，请使用此方法。

示例（请注意，这会显示经过模糊处理的字符串文本，以免公开帐户密钥或 SAS）：

`h"https://fabrikam.blob.core.chinacloudapi.cn/container/path/to/file.csv;<storage_account_key_text, ends with '=='>"`
`h"https://fabrikam.blob.core.chinacloudapi.cn/container/path/to/file.csv?sv=...&sp=rwd"` 

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="azure-data-lake-store-gen-2"></a>Azure Data Lake Store Gen 2

此提供程序支持访问 Azure Data Lake Store Gen 2 中的数据。

URI 的格式为：

`abfss://` *Filesystem* `@` *StorageAccountName* `.dfs.core.chinacloudapi.cn/` *Path* `;` *CallerCredentials*

其中：

* Filesystem 是 ADLS 文件系统对象（大致等效于 Blob 容器）的名称
* StorageAccountName 是存储帐户的名称
* Path 是所访问的目录或文件的路径，斜杠 (`/`) 字符用作分隔符。
* CallerCredentials 表示用于访问服务的凭据，如下所述。

访问 Azure Data Lake Store Gen 2 时，调用方必须提供用于访问服务的有效凭据。 支持以下提供凭据的方法：

* 将 `;sharedkey=`AccountKey 追加到 URI（其中 _AccountKey_ 为存储帐户密钥）
* 将 `;impersonate` 追加到 URI。 Kusto 将使用请求者的主体标识并模拟它来访问资源。 主体需要具有相应的 RBAC 角色分配才能执行读/写操作，如[此文](https://docs.azure.cn/storage/blobs/data-lake-storage-access-control)所述。 （例如，读取操作的最小角色是 `Storage Blob Data Reader` 角色）。
* 将 `;token=`AadToken 追加到 URI，其中的 AadToken 为 base-64 编码的 AAD 访问令牌（请确保该令牌适用于资源 `https://storage.azure.com/`）。
* 将 `;prompt` 追加到 URI。 Kusto 在需要访问资源时请求用户凭据。 （“提示用户”功能禁止用于云部署，仅在测试环境中启用。）
* 使用 Azure Data Lake Storage Gen 2 的标准查询 (`?sig=...`) 提供共享访问签名 (SAS) 密钥。 当 Kusto 需要在有限的时间内访问资源时，请使用此方法。


<!--
### Azure Data Lake Store Gen 1

This provider supports accessing files and directories in Azure Data Lake Store.
It must be provided with credentials (Kusto doesn't use its own AAD principal to
access Azure Data Lake.) The following methods of providing credentials are
supported:

* Append `;impersonate` to the URI. Kusto will use the requestor's principal
  identity and impersonate it to access the resource
* Append `;token=`*AadToken* to the URI, with *AadToken* being a base-64
  encoded AAD access token (make sure the token is for the resource `https://management.chinacloudapi.cn/`).
* Append `;prompt` to the URI. Kusto will request user credentials
  when it needs to access the resource. (Prompting the user is disabled for
  cloud deployments and is only enabled in test environments.)
-->
