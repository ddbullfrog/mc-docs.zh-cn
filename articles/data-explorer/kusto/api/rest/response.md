---
title: 查询/管理 HTTP 响应 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍了 Azure 数据资源管理器中的查询/管理 HTTP 响应。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 11/05/2018
ms.date: 10/29/2020
ms.openlocfilehash: cb67b826562d94fad946de8c4184a1ab79cb875f
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103898"
---
# <a name="querymanagement-http-response"></a>查询/管理 HTTP 响应

## <a name="response-status"></a>响应状态

HTTP 响应状态行遵循 HTTP 标准响应代码的要求。
例如，代码 200 指示成功。 

当前使用了以下状态代码，但可能会返回任何有效的 HTTP 代码。

|代码|子代码        |说明                                    |
|----|---------------|-----------------------------------------------|
|100 |继续       |客户端可以继续发送请求。       |
|200 |OK             |请求已成功开始处理。       |
|400 |BadRequest     |请求格式不正确且已（永久）失败。|
|401 |未授权   |客户端需要先进行身份验证。            |
|403 |禁止      |客户端请求被拒绝。                      |
|404 |NotFound       |请求引用了一个不存在的实体。      |
|413 |PayloadTooLarge|请求有效负载超出了限制。               |
|429 |TooManyRequests|由于限制，请求已被拒绝。 |
|504 |超时        |请求已超时。                         |
|520 |ServiceError   |服务处理请求时发现一个错误。|

> [!NOTE]
> 状态代码为 200 表明请求处理已成功开始，而不是已成功完成。
> 在处理请求期间在返回状态代码 200 之后发生的失败称为“部分查询失败”。在发生这种失败时，会将特殊指示符注入到响应流中，向客户端发出相应警报。

## <a name="response-headers"></a>响应标头

将返回以下自定义标头。

|自定义标头           |说明                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-client-request-id`|在具有相同名称的请求标头中发送的唯一请求标识符，或某个唯一标识符。     |
|`x-ms-activity-id`      |请求的全局唯一相关标识符。 它是由服务创建的。                    |

## <a name="response-body"></a>响应正文

如果状态代码为 200，则响应正文是一个 JSON 文档，该文档将查询或控制命令的结果编码为一个矩形表序列。
有关详细信息，请参阅下文。

> [!NOTE]
> 此表序列通过 SDK 反映。 例如，使用 .NET Framework Kusto.Data 库时，表的序列将成为 SDK 返回的 `System.Data.IDataReader` 对象中的结果。

如果状态代码指示了 401 之外的 4xx 或 5xx 错误，则响应正文是对失败详细信息进行了编码的 JSON 文档。
有关详细信息，请参阅 [Microsoft REST API 准则](https://github.com/microsoft/api-guidelines)。

> [!NOTE]
> 如果请求中未包含 `Accept` 标头，则失败的响应正文不一定是 JSON 文档。

## <a name="json-encoding-of-a-sequence-of-tables"></a>表序列的 JSON 编码

表序列的 JSON 编码是包含以下名称/值对的单个 JSON 属性包。

|名称  |Value                              |
|------|-----------------------------------|
|表|表属性包的数组。|

表属性包具有以下名称/值对。

|名称     |Value                               |
|---------|------------------------------------|
|TableName|用于标识表的字符串。 |
|列  |列属性包的数组。|
|“行”     |行数组的数组。          |

列属性包具有以下名称/值对。

|名称      |Value                                                          |
|----------|---------------------------------------------------------------|
|ColumnName|用于标识列的字符串。                           |
|数据类型  |一个字符串，提供列的大致 .NET 类型。|
|ColumnType|一个字符串，提供列的[标量数据类型](../../query/scalar-data-types/index.md)。|

行数组的顺序与相应列数组的顺序相同。
行数组还有一个与相关列的行值相符的元素。
不能用 JSON 表示的标量数据类型（例如 `datetime` 和 `timespan`）表示为 JSON 字符串。

下面的示例显示了一个可能的此类对象，其中包含单个名为 `Table_0` 的表，该表具有类型为 `string` 的单个列 `Text` 以及单个行。

```json
{
    "Tables": [{
        "TableName": "Table_0",
        "Columns": [{
            "ColumnName": "Text",
            "DataType": "String",
            "ColumnType": "string"
        }],
        "Rows": [["Hello, World!"]]
}
```

另一个示例： 

:::image type="content" source="../images/rest-json-representation.png" alt-text="此屏幕截图显示了一个 JSON 文件（包含一个表对象数组）的树视图。":::

## <a name="the-meaning-of-tables-in-the-response"></a>响应中的表的含义

在大多数情况下，控制命令返回包含单个表的结果，其中包含由控制命令生成的信息。 例如，`.show databases` 命令返回单个表，其中包含群集中所有可访问的数据库的详细信息。

查询通常会返回多个表。
对于每个[表格表达式语句](../../query/tabularexpressionstatements.md)，将按顺序生成一个或多个表，表示该语句产生的结果。

> [!NOTE]
> 由于存在[批次](../../query/batches.md)和[分支运算符](../../query/forkoperator.md)，可能会有多个这样的表。

通常会生成三个表：
* @ExtendedProperties 表，提供附加的值，例如客户端可视化效果说明。 例如，生成这些值来反映 [render 运算符](../../query/renderoperator.md)和[数据库游标](../../management/databasecursor.md)中的信息。
  
  此表包含 `string` 类型的单个列，其中包含类似于 JSON 的值：

  |Value|
  |-----|
  |{"Visualization":"piechart",...}|
  |{"Cursor":"637239957206013576"}|

* QueryStatus 表，提供有关查询本身的执行的附加信息（例如，它是否已成功完成）以及查询使用的资源。

  此表具有以下结构：

  |Timestamp                  |severity|SeverityName|StatusCode|StatusDescription            |计数|RequestId|ActivityId|SubActivityId|ClientActivityId|
  |---------------------------|--------|------------|----------|-----------------------------|-----|---------|----------|-------------|----------------|
  |2020-05-02 06:09:12.7052077|4       |信息        | 0        | 查询已成功完成|1    |...      |...       |...          |...             |

  严重性值为 2 或更小表示失败。

* TableOfContents 表是最后创建的，它列出结果中的其他表。 

  下面是此表的一个示例：

  |Ordinal|种类            |名称               |ID                                  |PrettyName|
  |-------|----------------|-------------------|------------------------------------|----------|
  |0      | QueryResult    |PrimaryResult      |db9520f9-0455-4cb5-b257-53068497605a||
  |1      | QueryProperties|@ExtendedProperties|908901f6-5319-4809-ae9e-009068c267c7||
  |2      | QueryStatus    |QueryStatus        |00000000-0000-0000-0000-000000000000||
