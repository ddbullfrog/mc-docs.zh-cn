---
title: Azure 数据工厂中的 Excel 格式
description: 本主题介绍了如何处理 Azure 数据工厂中的 Excel 格式。
author: WenJason
manager: digimobile
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
origin.date: 09/14/2020
ms.date: 10/19/2020
ms.author: v-jay
ms.openlocfilehash: 8fc362e86c292f5a1a55e0c60d7c5351121ee3a7
ms.sourcegitcommit: 6309f3a5d9506d45ef6352e0e14e75744c595898
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92121702"
---
# <a name="excel-format-in-azure-data-factory"></a>Azure 数据工厂中的 Excel 格式
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

如果要**分析 Excel 文件**，请按此文的要求操作。 Azure 数据工厂同时支持“.xls”和“.xlsx”。

以下连接器支持 Excel 格式：[Amazon S3](connector-amazon-simple-storage-service.md)、[Azure Blob](connector-azure-blob-storage.md)、[Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)、[Azure 文件存储](connector-azure-file-storage.md)、[文件系统](connector-file-system.md)、[FTP](connector-ftp.md)、[Google 云存储](connector-google-cloud-storage.md)、[HDFS](connector-hdfs.md)、[HTTP](connector-http.md) 和 [SFTP](connector-sftp.md)。 它可以作为源，但不可作为接收器。

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各部分和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)一文。 本部分提供 Excel 数据集支持的属性列表。

| 属性         | 说明                                                  | 必须 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 数据集的 type 属性必须设置为“Excel”。   | 是      |
| location         | 文件的位置设置。 每个基于文件的连接器在 `location` 下都有其自己的位置类型和支持的属性。 | 是      |
| sheetName        | 要读取数据的 Excel 工作表名称。                       | 是      |
| range            | 给定工作表中用于定位选择性数据的单元格范围，例如：<br>- 未指定：以表的形式从第一个非空行和列读取整个工作表<br>- `A3`：读取从给定单元格开始的表，动态检测下面的所有行和右侧的所有列<br>- `A3:H5`：以表的形式读取此固定范围<br>- `A3:A3`：读取此单个单元格 | 否       |
| firstRowAsHeader | 指定是否要将给定工作表/范围内的第一行视为带有列名的标题行。<br>允许的值为 **true** 和 **true**（默认值）。 | 否       |
| nullValue        | 指定 null 值的字符串表示形式。 <br>默认值为**空字符串**。 | 否       |
| compression | 用来配置文件压缩的属性组。 如果需要在活动执行期间进行压缩/解压缩，请配置此部分。 | 否 |
| type<br/>（在 `compression` 下） | 用来读取/写入 JSON 文件的压缩编解码器。 <br>允许的值为 bzip2、gzip、deflate、ZipDeflate、TarGzip、snappy 或 lz4      。 默认设置是不压缩。<br>请注意，复制活动当前不支持“snappy”和“lz4”。<br>**注意**，使用复制活动来解压缩 ZipDeflate 文件并写入到基于文件的接收器数据存储时，会将文件提取到以下文件夹：`<path specified in dataset>/<folder named as source zip file>/`。 | 否。  |
| level<br/>（在 `compression` 下） | 压缩率。 <br>允许的值为 **Optimal** 或 **Fastest**。<br>- **Fastest**：尽快完成压缩操作，不过，无法以最佳方式压缩生成的文件。<br>- **Optimal**：以最佳方式完成压缩操作，不过，需要耗费更长的时间。 有关详细信息，请参阅 [Compression Level](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx)（压缩级别）主题。 | 否       |

下面是 Azure Blob 存储上 Excel 数据集的示例：

```json
{
    "name": "ExcelDataset",
    "properties": {
        "type": "Excel",
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
            "sheetName": "MyWorksheet",
            "range": "A3:H5",
            "firstRowAsHeader": true
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各部分和属性的完整列表，请参阅[管道](concepts-pipelines-activities.md)一文。 本部分提供 Excel 源支持的属性列表。

### <a name="excel-as-source"></a>Excel 作为源 

复制活动的 ***\*source\**** 节支持以下属性。

| 属性      | 说明                                                  | 必须 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 复制活动源的 type 属性必须设置为“ExcelSource”。 | 是      |
| storeSettings | 有关如何从数据存储读取数据的一组属性。 每个基于文件的连接器在 `storeSettings` 下都有其自己支持的读取设置。 | 否       |

```json
"activities": [
    {
        "name": "CopyFromExcel",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "ExcelSource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            ...
        }
        ...
    }
]
```

## <a name="next-steps"></a>后续步骤

- [复制活动概述](copy-activity-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)
- [GetMetadata 活动](control-flow-get-metadata-activity.md)
