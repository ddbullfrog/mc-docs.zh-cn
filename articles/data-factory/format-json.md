---
title: Azure 数据工厂中的 JSON 格式
description: 本主题介绍了如何处理 Azure 数据工厂中的 JSON 格式。
author: WenJason
manager: digimobile
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
origin.date: 08/10/2020
ms.date: 09/21/2020
ms.author: v-jay
ms.openlocfilehash: 79bf555cd521d57c8b93ccaf33b07a9a23a15f8a
ms.sourcegitcommit: f5d53d42d58c76bb41da4ea1ff71e204e92ab1a7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90524055"
---
# <a name="json-format-in-azure-data-factory"></a>Azure 数据工厂中的 JSON 格式

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

如果要**分析 JSON 文件或以 JSON 格式写入数据**，请遵循本文中的说明。 

以下连接器支持 JSON 格式：[Amazon S3](connector-amazon-simple-storage-service.md)、[Azure Blob](connector-azure-blob-storage.md)、[Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)、[Azure 文件存储](connector-azure-file-storage.md)、[文件系统](connector-file-system.md)、[FTP](connector-ftp.md)、[Google 云存储](connector-google-cloud-storage.md)、[HDFS](connector-hdfs.md)、[HTTP](connector-http.md) 和 [SFTP](connector-sftp.md)。

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供 JSON 数据集支持的属性列表。

| 属性         | 说明                                                  | 必须 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 数据集的 type 属性必须设置为 **Json**。 | 是      |
| location         | 文件的位置设置。 每个基于文件的连接器在 `location` 下都有其自己的位置类型和支持的属性。 **请在连接器文章 -> 数据集属性部分中查看详细信息**。 | 是      |
| encodingName     | 用于读取/写入测试文件的编码类型。 <br>可用的值如下："UTF-8"、"UTF-16"、"UTF-16BE"、"UTF-32"、"UTF-32BE"、"US-ASCII"、"UTF-7"、"BIG5"、"EUC-JP"、"EUC-KR"、"GB2312"、"GB18030"、"JOHAB"、"SHIFT-JIS"、"CP875"、"CP866"、"IBM00858"、"IBM037"、"IBM273"、"IBM437"、"IBM500"、"IBM737"、"IBM775"、"IBM850"、"IBM852"、"IBM855"、"IBM857"、"IBM860"、"IBM861"、"IBM863"、"IBM864"、"IBM865"、"IBM869"、"IBM870"、"IBM01140"、"IBM01141"、"IBM01142"、"IBM01143"、"IBM01144"、"IBM01145"、"IBM01146"、"IBM01147"、"IBM01148"、"IBM01149"、"ISO-2022-JP"、"ISO-2022-KR"、"ISO-8859-1"、"ISO-8859-2"、"ISO-8859-3"、"ISO-8859-4"、"ISO-8859-5"、"ISO-8859-6"、"ISO-8859-7"、"ISO-8859-8"、"ISO-8859-9"、"ISO-8859-13"、"ISO-8859-15"、"WINDOWS-874"、"WINDOWS-1250"、"WINDOWS-1251"、"WINDOWS-1252"、"WINDOWS-1253"、"WINDOWS-1254"、"WINDOWS-1255"、"WINDOWS-1256"、"WINDOWS-1257"、"WINDOWS-1258"。| 否       |
| compression | 用来配置文件压缩的属性组。 如果需要在活动执行期间进行压缩/解压缩，请配置此部分。 | 否 |
| type<br/>（在 `compression` 下） | 用来读取/写入 JSON 文件的压缩编解码器。 <br>允许的值为 **bzip2**、**gzip**、**deflate**、**ZipDeflate**、**snappy** 或 **lz4**。 保存文件时使用。 默认设置是不压缩。<br>注意，复制活动当前不支持“snappy”和“lz4”。<br>**注意**，使用复制活动解压缩 ZipDeflate 文件并将其写入基于文件的接收器数据存储时，默认情况下文件将提取到 `<path specified in dataset>/<folder named as source zip file>/` 文件夹，对[复制活动源](#json-as-source)使用 `preserveZipFileNameAsFolder` 来控制是否以文件夹结构形式保留 zip 文件名。 | 否。  |
| level<br/>（在 `compression` 下） | 压缩率。 <br>允许的值为 **Optimal** 或 **Fastest**。<br>- **Fastest**：尽快完成压缩操作，不过，无法以最佳方式压缩生成的文件。<br>- **Optimal**：以最佳方式完成压缩操作，不过，需要耗费更长的时间。 有关详细信息，请参阅 [Compression Level](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx)（压缩级别）主题。 | 否       |

下面是 Azure Blob 存储上的 JSON 数据集的示例：

```json
{
    "name": "JSONDataset",
    "properties": {
        "type": "Json",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "gzip"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 JSON 源和接收器支持的属性列表。

若要了解如何从 JSON 文件中提取数据并将其映射到接收器数据存储/格式（反之亦然），请参阅[架构映射](copy-activity-schema-and-type-mapping.md)。

### <a name="json-as-source"></a>以 JSON 作为源

复制活动的 ***\*source\**** 节支持以下属性。

| 属性      | 描述                                                  | 必须 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 复制活动源的 type 属性必须设置为 **JSONSource**。 | 是      |
| formatSettings | 一组属性。 请参阅下面的“JSON 读取设置”表。 | 否       |
| storeSettings | 有关如何从数据存储读取数据的一组属性。 每个基于文件的连接器在 `storeSettings` 下都有其自己支持的读取设置。 **请在连接器文章 -> 复制活动属性部分中查看详细信息**。 | 否       |

`formatSettings` 下支持的“JSON 读取设置”：

| 属性      | 描述                                                  | 必须 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | formatSettings 的 type 必须设置为“JsonReadSettings”。 | 是      |
| compressionProperties | 一组属性，指示如何为给定的压缩编解码器解压缩数据。 | 否       |
| preserveZipFileNameAsFolder<br>（在 `compressionProperties` 下） | 当输入数据集配置了 ZipDeflate 压缩时适用。 指示是否在复制过程中以文件夹结构形式保留源 zip 文件名。<br>- 当设置为“true”（默认值）时，数据工厂会将已解压缩的文件写入 `<path specified in dataset>/<folder named as source zip file>/`。<br>- 当设置为“false”时，数据工厂会直接将未解压缩的文件写入 `<path specified in dataset>`。 请确保在不同的源 zip 文件中没有重复的文件名，以避免产生冲突或出现意外行为。  | 否 |

### <a name="json-as-sink"></a>JSON 作为接收器

复制活动的 ***\*sink\**** 节支持以下属性。

| 属性      | 说明                                                  | 必需 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 复制活动源的 type 属性必须设置为 **JSONSink**。 | 是      |
| formatSettings | 一组属性。 请参阅下面的“JSON 写入设置”表。 | 否       |
| storeSettings | 有关如何将数据写入到数据存储的一组属性。 每个基于文件的连接器在 `storeSettings` 下都有其自身支持的写入设置。 **请在连接器文章 -> 复制活动属性部分中查看详细信息**。 | 否       |

`formatSettings` 下支持的 **JSON 写入设置**：

| 属性      | 说明                                                  | 必需                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | formatSettings 的类型必须设置为 **JsonWriteSettings**。 | 是                                                   |
| filePattern |指示每个 JSON 文件中存储的数据模式。 允许的值为：setOfObjects (JSON Lines) 和 arrayOfObjects。 **默认**值为 **setOfObjects**。 请参阅 [JSON 文件模式](#json-file-patterns)部分，详细了解这些模式。 |否 |

### <a name="json-file-patterns"></a>JSON 文件模式

从 JSON 文件复制数据时，复制活动可自动检测并分析以下 JSON 文件模式。 将数据写入 JSON 文件时，可以在复制活动接收器上配置文件模式。

- **类型 I：setOfObjects**

    每个文件都包含单一对象、JSON Lines 或串联的对象。

    * **单一对象 JSON 示例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        ```

    * **JSON Lines（接收器的默认值）**

        ```json
        {"time":"2015-04-29T07:12:20.9100000Z","callingimsi":"466920403025604","callingnum1":"678948008","callingnum2":"567834760","switch1":"China","switch2":"Germany"}
        {"time":"2015-04-29T07:13:21.0220000Z","callingimsi":"466922202613463","callingnum1":"123436380","callingnum2":"789037573","switch1":"US","switch2":"UK"}
        {"time":"2015-04-29T07:13:21.4370000Z","callingimsi":"466923101048691","callingnum1":"678901578","callingnum2":"345626404","switch1":"Germany","switch2":"UK"}
        ```

    * **串连的 JSON 示例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        }
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
        ```

- **类型 II：arrayOfObjects**

    每个文件包含对象的数组。

    ```json
    [
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        },
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        },
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
    ]
    ```

## <a name="next-steps"></a>后续步骤

- [复制活动概述](copy-activity-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)
- [GetMetadata 活动](control-flow-get-metadata-activity.md)
