---
title: 查询参数声明语句 - Azure 数据资源管理器
description: 本文介绍了 Azure 数据资源管理器中的查询参数声明语句。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 10/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: cdaf44b2b540cb03305fd7b97ff1125d0f5d03e8
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104020"
---
# <a name="query-parameters-declaration-statement"></a>查询参数声明语句

::: zone pivot="azuredataexplorer"

发送到 Kusto 的查询可以包含一组名称或值对。 这些对与查询文本自身一起被称为查询参数。 查询可以通过在查询参数声明语句中指定名称和类型来引用一个或多个值。

查询参数有两个主要用途：

* 用作防范注入攻击的保护机制。
* 用作对查询进行参数化的方式。

具体而言，如果客户端应用程序在查询中组合了用户提供的输入，然后将查询发送到 Kusto，则客户端应用程序应使用该机制来防止与 SQL 注入攻击相当的 Kusto 攻击。

## <a name="declaring-query-parameters"></a>声明查询参数

若要引用查询参数、查询文本或它使用的函数，必须首先声明它使用哪个查询参数。 对于每个参数，声明都提供名称和标量类型。 此外，参数还可以具有默认值。 如果请求未提供参数的具体值，则会使用默认值。 然后，Kusto 根据该类型的常规分析规则来分析查询参数的值。

## <a name="syntax"></a>语法

`declare` `query_parameters` `(` *Name1* `:` *Type1* [`=` *DefaultValue1* ] [`,`...] `);`

* Name1：查询中使用的查询参数的名称。
* Type1：对应的类型，例如 `string` 或 `datetime`。
  用户提供的值将编码为字符串，Kusto 会将相应的分析方法应用于查询参数以获取强类型值。
* DefaultValue1：参数的可选默认值。 此值必须是合适标量类型的文本。

> [!NOTE]
> 与[用户定义的函数](functions/user-defined-functions.md)一样，`dynamic` 类型的查询参数不能具有默认值。

## <a name="examples"></a>示例

```kusto
declare query_parameters(UserName:string, Password:string);
print n=UserName, p=hash(Password)
```

```kusto
declare query_parameters(percentage:long = 90);
T | where Likelihood > percentage
```

## <a name="specifying-query-parameters-in-a-client-application"></a>在客户端应用程序中指定查询参数

查询参数的名称和值由进行查询的应用程序作为 `string` 值提供。 名称不可以重复。

值的解释根据查询参数声明语句来完成。 每个值都作为查询正文中的文本进行分析。 分析根据查询参数声明语句指定的类型来完成。

### <a name="rest-api"></a>REST API

查询参数由客户端应用程序通过请求正文的 JSON 对象的 `properties` 槽在名为 `Parameters` 的嵌套属性包中提供。 例如，下面是对 Kusto 的一个 REST API 调用的正文，该调用会计算某个用户的年龄（可能是采用让应用程序询问用户的生日的方式）。

``` json
{
    "ns": null,
    "db": "myDB",
    "csl": "declare query_parameters(birthday:datetime); print strcat(\"Your age is: \", tostring(now() - birthday))",
    "properties": "{\"Options\":{},\"Parameters\":{\"birthday\":\"datetime(1970-05-11)\",\"courses\":\"dynamic(['Java', 'C++'])\"}}"
}
```

### <a name="kusto-net-sdk"></a>Kusto .NET SDK

若要在使用 Kusto .NET 客户端库时提供查询参数的名称和值，可以创建 `ClientRequestProperties` 对象的一个新实例，然后使用 `HasParameter`、`SetParameter` 和 `ClearParameter` 方法来操作查询参数。 此类为 `SetParameter` 提供了许多强类型的重载；在内部，它们会生成相应的查询语言文本，并通过 REST API 将其作为 `string` 发送，如上所述。 查询文本自身仍需[声明查询参数](#declaring-query-parameters)。

### <a name="kustoexplorer"></a>Kusto.Explorer

若要设置向服务发出请求时发送的查询参数，请使用 **查询参数** “扳手”图标 (`ALT` + `P`)。

::: zone-end

::: zone pivot="azuremonitor"

Azure Monitor 不支持此功能

::: zone-end
