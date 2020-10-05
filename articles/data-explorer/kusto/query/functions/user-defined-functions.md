---
title: 用户定义的函数 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中用户定义的函数。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 03/12/2020
ms.date: 09/24/2020
ms.openlocfilehash: fe6bec0f3cb7ece097bf6e6faa7099aacba03f17
ms.sourcegitcommit: f3fee8e6a52e3d8a5bd3cf240410ddc8c09abac9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/24/2020
ms.locfileid: "91146420"
---
# <a name="user-defined-functions"></a>用户定义的函数

“用户定义的函数”是可重复使用的子查询，可将其定义为查询本身（临时函数）的一部分，或作为数据库元数据（存储函数）的一部分长期保存  。 通过“名称”调用用户定义的函数，为其提供零个或多个“输入参数”（可以是标量或表格），这些函数将根据函数体生成单个值（可以是标量或表格）  。

用户定义的函数属于以下两个类别之一：

* 标量函数 
* 表格函数 

函数的输入参数和输出决定它是标量还是表格，进而确定其使用方式。 

## <a name="scalar-function"></a>标量函数

* 具有零个输入参数，或者其所有输入参数均为标量值
* 生成单个标量值
* 可在允许使用标量表达式的任何地方使用
* 只能使用定义它的行上下文
* 只能引用可访问架构中的表（和视图）

## <a name="tabular-function"></a>表格函数

* 接受一个或多个表格输入参数、零个或多个标量输入参数和/或：
* 生成单个表格值

## <a name="function-names"></a>函数名称

有效的用户定义的函数名称必须遵循与其他实体相同的[标识符命名规则](../schema-entities/entity-names.md#identifier-naming-rules)。

该名称在定义的范围内也必须是唯一的。

## <a name="input-arguments"></a>输入参数

有效的用户定义的函数遵循以下规则：

* 用户定义的函数具有一个包含零个或多个输入参数的强类型列表。
* 输入参数具有名称、类型和（对于标量参数）[默认值](#default-values)。
* 输入参数的名称是一个标识符。
* 输入参数的类型为标量数据类型之一或表格架构。

语法上，输入参数列表是用逗号分隔的参数定义列表，并用括号括起来。 每个参数定义都指定为

```
ArgName:ArgType [= ArgDefaultValue]
```
 对于表格参数，ArgType 具有与表定义相同的语法（括号和列名/类型对列表），另外还支持单独的 `(*)`（指示“任何表格架构）。

例如：

|语法                        |输入参数列表说明                                 |
|------------------------------|-----------------------------------------------------------------|
|`()`                          |无参数|
|`(s:string)`                  |名为 `s` 的单个标量参数，其值类型为 `string`|
|`(a:long, b:bool=true)`       |两个标量参数，其中的第二个参数具有默认值    |
|`(T1:(*), T2(r:real), b:bool)`|三个参数（两个表格参数和一个标量参数）  |

> [!NOTE]
> 同时使用表格输入参数和标量输入参数时，请将所有表格输入参数置于标量输入参数之前。

## <a name="examples"></a>示例

标量函数：

```kusto
let Add7 = (arg0:long = 5) { arg0 + 7 };
range x from 1 to 10 step 1
| extend x_plus_7 = Add7(x), five_plus_seven = Add7()
```

不带参数的表格函数：

```kusto
let tenNumbers = () { range x from 1 to 10 step 1};
tenNumbers
| extend x_plus_7 = x + 7
```

同时采用表格输入和标量输入的表格函数：

```kusto
let MyFilter = (T:(x:long), v:long) {
  T | where x >= v
};
MyFilter((range x from 1 to 10 step 1), 9)
```

|x|
|---|
|9|
|10|

使用未指定列的表格输入的表格函数。
可将任意表传递给函数，并且不能在函数内引用任何表列。

```kusto
let MyDistinct = (T:(*)) {
  T | distinct *
};
MyDistinct((range x from 1 to 3 step 1))
```

|x|
|---|
|1|
|2|
|3|

## <a name="declaring-user-defined-functions"></a>声明用户定义的函数

用户定义的函数的声明提供：

* 函数名称
* 函数架构（它接受的参数，如果有）
* 函数体

> [!Note]
> 不支持重载函数。 不能创建多个具有相同名称和不同输入架构的函数。

> [!TIP]
> Lambda 函数没有名称，该函数使用 [let 语句](../letstatement.md)绑定到名称。 因此，可将其视为用户定义的存储函数。
> 示例：Lambda 函数的声明，该函数接受两个参数（称为 `s` 的 `string` 和称为 `i` 的 `long`）。 它将返回第一个（将其转换为数字后）和第二个的乘积。 Lambda 绑定到名称 `f`：

```kusto
let f=(s:string, i:long) {
    tolong(s) * i
};
```

函数体包括：

* 恰好一个表达式，该表达式提供函数的返回值（标量或表格值）。
* 任何数量（零个或多个）的 [let 语句](../letstatement.md)，这些语句的范围是函数体的范围。 如果指定，则 let 语句必须在定义函数返回值的表达式之前。
* 任何数量（零个或多个）的[查询参数语句](../queryparametersstatement.md)，这些语句声明函数使用的查询参数。 如果指定，则这些语句必须在定义函数返回值的表达式之前。

> [!NOTE]
> 函数体内不支持“顶级”查询所支持的其他类型的[查询语句](../statements.md)。

### <a name="examples-of-user-defined-functions"></a>用户定义的函数示例 

**使用 let 语句的用户定义的函数**

下面的示例将名称 `Test` 绑定到使用三个 let 语句的用户定义的函数 (Lambda)。 输出为 `70`：

```kusto
let Test1 = (id: int) {
  let Test2 = 10;
  let Test3 = 10 + Test2 + id;
  let Test4 = (arg: int) {
      let Test5 = 20;
      Test2 + Test3 + Test5 + arg
  };
  Test4(10)
};
range x from 1 to Test1(10) step 1
| count
```

**定义参数默认值的用户定义的函数**

以下示例演示一个接受三个参数的函数。 后两个参数具有默认值，并且不必存在于调用站点。

```kusto
let f = (a:long, b:string = "b.default", c:long = 0) {
  strcat(a, "-", b, "-", c)
};
print f(12, c=7) // Returns "12-b.default-7"
```

## <a name="invoking-a-user-defined-function"></a>调用用户定义的函数

不带参数的用户定义的函数可以通过函数名称或名称以及括号中的空参数列表进行调用。

示例:

```kusto
// Bind the identifier a to a user-defined function (lambda) that takes
// no arguments and returns a constant of type long:
let a=(){123};
// Invoke the function in two equivalent ways:
range x from 1 to 10 step 1
| extend y = x * a, z = x * a() 
```

```kusto
// Bind the identifier T to a user-defined function (lambda) that takes
// no arguments and returns a random two-by-two table:
let T=(){
  range x from 1 to 2 step 1
  | project x1 = rand(), x2 = rand()
};
// Invoke the function in two equivalent ways:
// (Note that the second invocation must be itself wrapped in
// an additional set of parentheses, as the union operator
// differentiates between "plain" names and expressions)
union T, (T())
```

采用一个或多个标量参数的用户定义的函数可以使用表名称和括号中的具体参数列表进行调用：

```kusto
let f=(a:string, b:string) {
  strcat(a, " (la la la)", b)
};
print f("hello", "world")
```

采用一个或多个表参数（和任意数量的标量参数）的用户定义的函数可以使用表名称和括号中的具体参数列表及进行调用：

```kusto
let MyFilter = (T:(x:long), v:long) {
  T | where x >= v 
};
MyFilter((range x from 1 to 10 step 1), 9)
```

还可以使用运算符 `invoke` 来调用采用一个或多个表参数并返回一个表的用户定义的函数。 如果函数的第一个具体表参数是 `invoke` 运算符的源，此函数很有用：

```kusto
let append_to_column_a=(T:(a:string), what:string) {
    T | extend a=strcat(a, " ", what)
};
datatable (a:string) ["sad", "really", "sad"]
| invoke append_to_column_a(":-)")
```

## <a name="default-values"></a>默认值

在以下条件下，函数可以为其某些参数提供默认值：

* 只能为标量参数提供默认值。
* 默认值始终为文本（常量）。 它们不能是任意计算。
* 没有默认值的参数始终位于具有默认值的参数之前。
* 调用方必须提供所有参数的值，并且没有以与函数声明相同的顺序进行排列的默认值。
* 调用方不需要为具有默认值的参数提供值，但可以这样做。
* 调用方可以按与参数顺序不匹配的顺序提供参数。 如果是这样，则他们必须为其参数命名。

以下示例返回一个包含两个相同记录的表。 在 `f` 的第一次调用中，参数被完全“打乱”，因此每个参数都被明确赋予了一个名称：

```kusto
let f = (a:long, b:string = "b.default", c:long = 0) {
  strcat(a, "-", b, "-", c)
};
union
  (print x=f(c=7, a=12)), // "12-b.default-7"
  (print x=f(12, c=7))    // "12-b.default-7"
```

|x|
|---|
|12-b.default-7|
|12-b.default-7|

## <a name="view-functions"></a>视图函数

如果用户定义的函数不采用任何参数并返回一个表格表达式，则可以将其标记为“视图”。 将用户定义的函数标记为视图意味着每当通配符表名称解析完成时，该函数的行为就类似于表。
以下示例演示两个用户定义的函数（`T_view` 和 `T_notview`），并展示 `union` 中的通配符引用如何仅解析第一个函数：

```kusto
let T_view = view () { print x=1 };
let T_notview = () { print x=2 };
union T*
```

## <a name="restrictions"></a>限制

存在以下限制：

* 用户定义的函数无法传入 [toscalar()](../toscalarfunction.md) 调用信息中，该信息取决于调用该函数的行上下文。
* 不能使用随行上下文而变化的参数来调用返回表格表达式的用户定义的函数。
* 无法在远程群集上调用至少具有一个表格输入的函数。
* 无法在远程群集上调用标量函数。

只有当用户定义的函数仅由标量函数构成，且不使用 `toscalar()` 时，才可以使用随行上下文而变化的参数来调用用户定义的函数。

**限制示例 1**

```kusto
// Supported:
// f is a scalar function that doesn't reference any tabular expression
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { now() + hours*1h };
Table2 | where Column != 123 | project d = f(10)

// Supported:
// f is a scalar function that references the tabular expression Table1,
// but is invoked with no reference to the current row context f(10):
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { toscalar(Table1 | summarize min(xdate) - hours*1h) };
Table2 | where Column != 123 | project d = f(10)

// Not supported:
// f is a scalar function that references the tabular expression Table1,
// and is invoked with a reference to the current row context f(Column):
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { toscalar(Table1 | summarize min(xdate) - hours*1h) };
Table2 | where Column != 123 | project d = f(Column)
```

**限制示例 2**

```kusto
// Not supported:
// f is a tabular function that is invoked in a context
// that expects a scalar value.
let Table1 = datatable(xdate:datetime)[datetime(1970-01-01)];
let Table2 = datatable(Column:long)[1235];
let f = (hours:long) { range x from 1 to hours step 1 | summarize make_list(x) };
Table2 | where Column != 123 | project d = f(Column)
```

## <a name="features-that-are-currently-unsupported-by-user-defined-functions"></a>用户定义的函数当前不支持的功能

为了确保完整性，下面列出了用户定义的函数当前不支持的一些常见请求功能：

1.  函数重载：目前没有办法重载函数（即创建多个具有相同名称和不同输入架构的函数）。

2.  默认值：函数的标量参数默认值必须是标量文本（常量）。 此外，存储函数的默认值不能为 `dynamic` 类型。
