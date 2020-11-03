---
title: geo_point_in_circle() - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的 geo_point_in_circle()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
origin.date: 02/03/2020
ms.date: 10/29/2020
ms.openlocfilehash: 7780174d442fbd28b92f75910ecd9426936e4de0
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106346"
---
# <a name="geo_point_in_circle"></a>geo_point_in_circle()

计算地理空间坐标是否在地球上的某个圆圈范围内。

## <a name="syntax"></a>语法

`geo_point_in_circle(`p_longitude`, `p_latitude`, `pc_longitude`, `pc_latitude`, `c_radius`)`

## <a name="arguments"></a>参数

* p_longitude：地理空间坐标经度值（度）。 有效值为 [-180, +180] 范围内的实数。
* p_latitude：地理空间坐标纬度值（度）。 有效值为 [-90, +90] 范围内的实数。
* pc_longitude：圆心地理空间坐标经度值（度）。 有效值为 [-180, +180] 范围内的实数。
* pc_latitude：圆心地理空间坐标纬度值（度）。 有效值为 [-90, +90] 范围内的实数。
* c_radius：以米为单位的圆半径。 有效值必须为正。

## <a name="returns"></a>返回

指示地理空间坐标是否在某个圆圈范围内。 如果坐标或圆圈无效，则查询将生成 null 结果。

> [!NOTE]
>* 按照 [WGS-84](https://earth-info.nga.mil/GandG/update/index.php?action=home) 坐标参考系统表示的地理空间坐标进行解释。
>* 用来测量地球上距离的大地测量基准是一个球体。
>* 圆圈是地球上的球冠。 球冠的半径沿球的曲面进行测量。

## <a name="examples"></a>示例

以下查询会查找由以下圆圈定义的区域中的所有位置：半径为 18 km，中心处于 [-122.317404, 47.609119] 坐标。

:::image type="content" source="images/geo-point-in-circle-function/circle-seattle.png" alt-text="西雅图附近的位置":::

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
datatable(longitude:real, latitude:real, place:string)
[
    real(-122.317404), 47.609119, 'Seattle',                   // In circle 
    real(-123.497688), 47.458098, 'Olympic National Forest',   // In exterior of circle  
    real(-122.201741), 47.677084, 'Kirkland',                  // In circle
    real(-122.443663), 47.247092, 'Tacoma',                    // In exterior of circle
    real(-122.121975), 47.671345, 'Redmond',                   // In circle
]
| where geo_point_in_circle(longitude, latitude, -122.317404, 47.609119, 18000)
| project place
```

|place|
|---|
|西雅图|
|Kirkland|
|Redmond|

奥兰多的暴风雨事件。 事件在奥兰多坐标内按 100 km 进行筛选，并且按事件类型和哈希进行聚合。

:::image type="content" source="images/geo-point-in-circle-function/orlando-storm-events.png" alt-text="西雅图附近的位置":::

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
StormEvents
| project BeginLon, BeginLat, EventType
| where geo_point_in_circle(BeginLon, BeginLat, real(-81.3891), 28.5346, 1000 * 100)
| summarize count() by EventType, hash = geo_point_to_s2cell(BeginLon, BeginLat)
| project geo_s2cell_to_central_point(hash), EventType, count_
| render piechart with (kind=map) // map rendering available in Kusto Explorer desktop
```

以下示例显示在特定位置 10 米内的纽约出租车载客。 相关载客按哈希进行聚合。

:::image type="content" source="images/geo-point-in-circle-function/circle-junction.png" alt-text="西雅图附近的位置":::

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
nyc_taxi
| project pickup_longitude, pickup_latitude
| where geo_point_in_circle( pickup_longitude, pickup_latitude, real(-73.9928), 40.7429, 10)
| summarize by hash = geo_point_to_s2cell(pickup_longitude, pickup_latitude, 22)
| project geo_s2cell_to_central_point(hash)
| render scatterchart with (kind = map) // map rendering available in Kusto Explorer desktop
```

以下示例会返回 true。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print in_circle = geo_point_in_circle(-122.143564, 47.535677, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|1|

以下示例会返回 false。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print in_circle = geo_point_in_circle(-122.137575, 47.630683, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|0|

以下示例由于坐标输入无效而返回 null 结果。

<!-- csl: https://help.kusto.chinacloudapi.cn/Samples -->
```kusto
print in_circle = geo_point_in_circle(200, 1, 1, 1, 1)
```

|in_circle|
|---|
||

以下示例由于圆半径输入无效而返回 null 结果。

```kusto
print in_circle = geo_point_in_circle(1, 1, 1, 1, -1)
```

|in_circle|
|---|
||
