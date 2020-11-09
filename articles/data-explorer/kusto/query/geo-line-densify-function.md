---
title: geo_line_densify() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 geo_line_densify()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
origin.date: 07/01/2020
ms.date: 10/29/2020
ms.openlocfilehash: f5f64f7b6de1a6d3da9c44d022b285f11c4e8af7
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104805"
---
# <a name="geo_line_densify"></a>geo_line_densify()

通过添加中间点将平面线或多线边缘转换为测地线。

## <a name="syntax"></a>语法

`geo_line_densify(`*lineString*`, `*tolerance*`)`

## <a name="arguments"></a>参数

* lineString：采用 [GeoJSON 格式](https://tools.ietf.org/html/rfc7946)，数据类型为[动态](./scalar-data-types/dynamic.md)的一条直线或多条直线。
* tolerance：一个可选数值，用于定义原始平面边缘与转换后的测地线边缘链之间的最大距离（以米为单位）。 支持的值范围为 [0.1, 10000]。 如果未指定，则使用默认值 `10`。

## <a name="returns"></a>返回

采用 [GeoJSON 格式](https://tools.ietf.org/html/rfc7946)的 [dynamic](./scalar-data-types/dynamic.md) 数据类型的致密线。 如果线或公差无效，则查询会生成 null 结果。

> [!NOTE]
> * 对按照 [WGS-84](https://earth-info.nga.mil/GandG/update/index.php?action=home) 坐标参考系统表示的地理空间坐标进行解释。

**LineString 定义**

dynamic({"type":"LineString","coordinates": [ [lng_1,lat_1], [lng_2,lat_2] ,..., [lng_N,lat_N] ]})

dynamic({"type":"MultiLineString","coordinates": [ [ line_1, line_2 ,..., line_N ] ]})

* LineString 坐标数组必须至少包含两个条目。
* 坐标 [经度,纬度] 必须有效。 经度必须是 [-180, +180] 范围内的实数，纬度必须是 [-90, +90] 范围内的实数。
* 边缘长度必须小于 180 度。 将选择两个顶点之间的最短边缘。

**约束**

* 致密线的最大点数限制为 10485760。
* 以 [dynamic](./scalar-data-types/dynamic.md) 格式存储线有大小限制。

**动机**

* [GeoJSON 格式](https://tools.ietf.org/html/rfc7946)将两点之间的边定义为直笛卡尔线。
* 是使用测地线还是平面边缘可能取决于数据集，在长边中尤其如此。

## <a name="examples"></a>示例

以下示例致密化曼哈顿岛的一条道路。 该边缘较短，平面边缘与其对应的测地线之间的距离小于公差规定的距离。 因此，结果保持不变。

```kusto
print densified_line = tostring(geo_line_densify(dynamic({"type":"LineString","coordinates":[[-73.949247, 40.796860],[-73.973017, 40.764323]]})))
```

|densified_line|
|---|
|{"type":"LineString","coordinates":[[-73.949247, 40.796860], [-73.973017, 40.764323]]}|

以下示例致密化长度约为 130 公里的边缘

```kusto
print densified_line = tostring(geo_line_densify(dynamic({"type":"LineString","coordinates":[[50, 50], [51, 51]]})))
```

|densified_line|
|---|
|{"type":"LineString","coordinates":[[50,50],[50.125,50.125],[50.25,50.25],[50.375,50.375],[50.5,50.5],[50.625,50.625],[50.75,50.75],[50.875,50.875],[51,51]]}|

下面的示例由于坐标输入无效而返回 null 结果。

```kusto
print densified_line = geo_line_densify(dynamic({"type":"LineString","coordinates":[[300,1],[1,1]]}))
```

|densified_line|
|---|
||

下面的示例由于公差输入无效而返回 null 结果。

```kusto
print densified_line = geo_line_densify(dynamic({"type":"LineString","coordinates":[[1,1],[2,2]]}), 0)
```

|densified_line|
|---|
||
