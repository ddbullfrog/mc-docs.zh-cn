---
title: 将示例数据引入 Azure 数据资源管理器
description: 了解如何将与天气相关的示例数据引入（加载）到 Azure 数据资源管理器。
author: orspod
ms.author: v-tawe
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: quickstart
origin.date: 08/12/2019
ms.date: 09/24/2020
ms.openlocfilehash: 121958b2b43b852d62a450ff20670c95c0dc5878
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146180"
---
# <a name="quickstart-ingest-sample-data-into-azure-data-explorer"></a>快速入门：将示例数据引入 Azure 数据资源管理器

本文介绍如何将示例数据引入（加载）到 Azure 数据资源管理器数据库。 有[多种方法可以引入数据](ingest-data-overview.md)；本文重点介绍适用于测试目的的基本方法。

> [!NOTE]
> 如果你已完成[使用 Azure 数据资源管理器 Python 库引入数据](python-ingest-data.md)，则已拥有此数据。

## <a name="prerequisites"></a>先决条件

[测试群集和数据库](create-cluster-database-portal.md)

## <a name="ingest-data"></a>引入数据

StormEvents  示例数据集包含[美国国家环境信息中心](https://www.ncdc.noaa.gov/stormevents/)中与天气相关的数据。

1. 登录到 [https://dataexplorer.azure.cn](https://dataexplorer.azure.cn)。

1. 在应用程序的左上角，选择“添加群集”  。

1. 在“添加群集”  对话框中，以 `https://<ClusterName>.<Region>.kusto.chinacloudapi.cn/` 格式输入群集 URL，然后选择“添加”  。

1. 粘贴以下命令，然后选择“运行”  以创建 StormEvents 表。

    ```Kusto
    .create table StormEvents (StartTime: datetime, EndTime: datetime, EpisodeId: int, EventId: int, State: string, EventType: string, InjuriesDirect: int, InjuriesIndirect: int, DeathsDirect: int, DeathsIndirect: int, DamageProperty: int, DamageCrops: int, Source: string, BeginLocation: string, EndLocation: string, BeginLat: real, BeginLon: real, EndLat: real, EndLon: real, EpisodeNarrative: string, EventNarrative: string, StormSummary: dynamic)
    ```
1. 粘贴以下命令，然后选择“运行”  以将数据引入到 StormEvents 表中。

    ```Kusto
    .ingest into table StormEvents h'https://kustosamplefiles.blob.core.chinacloudapi.cn/samplefiles/StormEvents.csv?sv=2019-12-12&ss=b&srt=o&sp=r&se=2022-09-05T02:23:52Z&st=2020-09-04T18:23:52Z&spr=https&sig=VrOfQMT1gUrHltJ8uhjYcCequEcfhjyyMX%2FSc3xsCy4%3D' with (ignoreFirstRecord=true)
    ```

1. 引入完成后，粘贴到以下查询中，在窗口中，选择该查询，并选择“运行”  。

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | take 10
    ```
    查询从引入的示例数据返回以下结果。

    ![查询结果](media/ingest-sample-data/query-results.png)

## <a name="next-steps"></a>后续步骤

* 请参阅 [Azure 数据资源管理器数据引入](ingest-data-overview.md)详细了解引入方法。
* [快速入门：在 Azure 数据资源管理器 Web UI 中查询数据](web-query-data.md)。
* 使用 Kusto 查询语言[编写查询](write-queries.md)。
