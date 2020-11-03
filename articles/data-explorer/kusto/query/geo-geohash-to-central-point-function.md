---
title: geo_geohash_to_central_point() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 geo_geohash_to_central_point()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
origin.date: 01/27/2020
ms.date: 10/29/2020
ms.openlocfilehash: 3c72d4b6c92c8adf7da1f8bc005579aa2b93f7c3
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106148"
---
# <a name="geo_geohash_to_central_point"></a>geo_geohash_to_central_point()

计算表示 Geohash 矩形区中心的地理空间坐标。


## <a name="syntax"></a>语法

`geo_geohash_to_central_point(`geohash`)`

## <a name="arguments"></a>参数

geohash：由 [geo_point_to_geohash()](geo-point-to-geohash-function.md) 计算得出的 Geohash 字符串值。 Geohash 字符串的长度可以是 1 到 18 个字符。

## <a name="returns"></a>返回

地理空间坐标值采用 [GeoJSON 格式](https://tools.ietf.org/html/rfc7946)，属于[动态](./scalar-data-types/dynamic.md)数据类型。 如果 geohash 无效，则查询会生成 null 结果。

> [!NOTE]
> GeoJSON 格式先指定经度，再指定纬度。

## <a name="examples"></a>示例

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print point = geo_geohash_to_central_point("sunny")
| extend coordinates = point.coordinates
| extend longitude = coordinates[0], latitude = coordinates[1]
```

|point|坐标|longitude|latitude|
|---|---|---|---|
|{<br>  "type":"Point",<br>  "coordinates": [<br>    42.47314453125,<br>    23.70849609375<br>  ]<br>}|[<br>  42.47314453125,<br>  23.70849609375<br>]|42.47314453125|23.70849609375|

下面的示例由于 geohash 输入无效而返回 null 结果。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print geohash = geo_geohash_to_central_point("a")
```

|geohash|
|---|
||

## <a name="example-creating-location-deep-links-for-bing-maps"></a>示例：为必应地图创建位置深层链接

通过指向 geohash 中心点，可以使用 geohash 值创建指向必应地图的深层链接 URL：

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
// Use string concatenation to create Bing Map deep-link URL from a geo-point
let point_to_map_url = (_point:dynamic, _title:string) 
{
    strcat('https://www.bing.com/maps?sp=point.', _point.coordinates[1] ,'_', _point.coordinates[0], '_', url_encode(_title)) 
};
// Convert geohash to center point, and then use 'point_to_map_url' to create Bing Map deep-link
let geohash_to_map_url = (_geohash:string, _title:string)
{
    point_to_map_url(geo_geohash_to_central_point(_geohash), _title)
};
print geohash = 'sv8wzvy7'
| extend url = geohash_to_map_url(geohash, "You are here")
```

|geohash|url|
|---|---|
|sv8wzvy7|[https://www.bing.com/maps?sp=point.32.15620994567871_34.80245590209961_You+are+here](https://www.bing.com/maps?sp=point.32.15620994567871_34.80245590209961_You+are+here)|
