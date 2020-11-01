---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 02/25/2020
title: 函数 - Azure Databricks
description: 了解 Azure Databricks 中 Apache Spark SQL 语言的各种内置函数的语法。
ms.openlocfilehash: 7cdea6cfe68beebc7e22309101af58f27e36dd6d
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472754"
---
# <a name="functions"></a>函数

本文列出了 Apache Spark SQL 中的内置函数。

## <a name=""></a><a id="f1"> </a>!

! expr - 逻辑非。

## <a name=""></a><a id="f2"> </a>%

expr1 % expr2 - 返回在进行 `expr1`/`expr2` 运算后的余数。

**示例：**

```sql
> SELECT 2 % 1.8;
 0.2
> SELECT MOD(2, 1.8);
 0.2
```

## <a name=""></a><a id="f3"> </a>&

expr1 & expr2 - 返回对 `expr1` 和 `expr2` 进行“按位与”运算后的结果。

**示例：**

```sql
> SELECT 3 & 5;
 1
```

## <a name=""></a><a id="f4"> </a>*

expr1 _ expr2 - 返回 `expr1`_`expr2`。

**示例：**

```sql
> SELECT 2 * 3;
 6
```

## <a name=""></a><a id="f5"> </a>+

expr1 + expr2 - 返回 `expr1`+`expr2`。

**示例：**

```sql
> SELECT 1 + 2;
 3
```

## <a name="-"></a><a id="-"> </a><a id="f-"> </a>-

expr1 - expr2 - 返回 `expr1`-`expr2`。

**示例：**

```sql
> SELECT 2 - 1;
 1
```

## <a name=""></a><a id="f6"> </a>/

expr1 / expr2 - 返回 `expr1`/`expr2`。 它始终执行浮点除法运算。

**示例：**

```sql
> SELECT 3 / 2;
 1.5
> SELECT 2L / 2L;
 1.0
```

## <a name=""></a><a id="f7"> </a><

expr1 < expr2 - 如果 `expr1` 小于 `expr2`，则返回 true。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以排序的类型。 例如，映射类型不可排序，因此不受支持。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 1 < 2;
 true
> SELECT 1.1 < '1';
 false
> SELECT to_date('2009-07-30 04:17:52') < to_date('2009-07-30 04:17:52');
 false
> SELECT to_date('2009-07-30 04:17:52') < to_date('2009-08-01 04:17:52');
 true
> SELECT 1 < NULL;
 NULL
```

## <a name=""></a><a id="f8"> </a><=

expr1 <= expr2 - 如果 `expr1` 小于或等于 `expr2`，则返回 true。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以排序的类型。 例如，映射类型不可排序，因此不受支持。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 <= 2;
 true
> SELECT 1.0 <= '1';
 true
> SELECT to_date('2009-07-30 04:17:52') <= to_date('2009-07-30 04:17:52');
 true
> SELECT to_date('2009-07-30 04:17:52') <= to_date('2009-08-01 04:17:52');
 true
> SELECT 1 <= NULL;
 NULL
```

## <a name=""></a><a id="f9"> </a><=>

expr1 <=> expr2 - 对于非 null 操作数，则返回与 EQUAL(=) 运算符相同的结果，但如果 expr1 和 expr2 都为 null，则返回 true；如果其中一个为 null，则返回 false。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以在等式比较中使用的类型。 不支持映射类型。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 <=> 2;
 true
> SELECT 1 <=> '1';
 true
> SELECT true <=> NULL;
 false
> SELECT NULL <=> NULL;
 true
```

## <a name=""></a><a id="f10"> </a>=

expr1 = expr2 - 如果 `expr1` 等于 `expr2`，则返回 true，否则返回 false。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以在等式比较中使用的类型。 不支持映射类型。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 = 2;
 true
> SELECT 1 = '1';
 true
> SELECT true = NULL;
 NULL
> SELECT NULL = NULL;
 NULL
```

## <a name=""></a><a id="f11"> </a>==

expr1 == expr2 - 如果 `expr1` 等于 `expr2`，则返回 true，否则返回 false。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以在等式比较中使用的类型。 不支持映射类型。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 == 2;
 true
> SELECT 1 == '1';
 true
> SELECT true == NULL;
 NULL
> SELECT NULL == NULL;
 NULL
```

## <a name=""></a><a id="f12"> </a>>

expr1 > expr2 - 如果 `expr1` 大于 `expr2`，则返回 true。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以排序的类型。 例如，映射类型不可排序，因此不受支持。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 > 1;
 true
> SELECT 2 > '1.1';
 true
> SELECT to_date('2009-07-30 04:17:52') > to_date('2009-07-30 04:17:52');
 false
> SELECT to_date('2009-07-30 04:17:52') > to_date('2009-08-01 04:17:52');
 false
> SELECT 1 > NULL;
 NULL
```

## <a name=""></a><a id="f13"> </a>>=

expr1 >= expr2 - 如果 `expr1` 大于或等于 `expr2`，则返回 true。

**参数：**

* expr1、expr2 - 这两个表达式必须是相同的类型或可以强制转换为一个通用类型，并且必须是可以排序的类型。 例如，映射类型不可排序，因此不受支持。 对于数组/结构等复杂类型，字段的数据类型必须是可以排序的。

**示例：**

```sql
> SELECT 2 >= 1;
 true
> SELECT 2.0 >= '2.1';
 false
> SELECT to_date('2009-07-30 04:17:52') >= to_date('2009-07-30 04:17:52');
 true
> SELECT to_date('2009-07-30 04:17:52') >= to_date('2009-08-01 04:17:52');
 false
> SELECT 1 >= NULL;
 NULL
```

## <a name=""></a><a id="f14"> </a>^

expr1 ^ expr2 - 返回对 `expr1` 和 `expr2` 进行“按位异或”运算的结果。

**示例：**

```sql
> SELECT 3 ^ 5;
 2
```

## <a name="abs"></a>abs

abs(expr) - 返回数值的绝对值。

**示例：**

```sql
> SELECT abs(-1);
 1
```

## <a name="acos"></a>acos

acos(expr) - 返回 `expr` 的反余弦，就像通过 `java.lang.Math.acos` 计算得出的一样。

**示例：**

```sql
> SELECT acos(1);
 0.0
> SELECT acos(2);
 NaN
```

## <a name="add_months"></a>add_months

add_months(start_date, num_months) - 返回 `start_date` 之后 `num_months` 个月的日期。

**示例：**

```sql
> SELECT add_months('2016-08-31', 1);
 2016-09-30
```

**最低生效版本：** 1.5.0

## <a name="aggregate"></a>aggregate

aggregate(expr, start, merge, finish) - 将二元运算符应用于初始状态和数组中的所有元素，并将其约简为单一状态。 通过应用完成函数将最终状态转换为最终结果。

**示例：**

```sql
> SELECT aggregate(array(1, 2, 3), 0, (acc, x) -> acc + x);
 6
> SELECT aggregate(array(1, 2, 3), 0, (acc, x) -> acc + x, acc -> acc * 10);
 60
```

**最低生效版本：** 2.4.0

## <a name="and"></a>and

expr1 and expr2 - 逻辑与。

## <a name="approx_count_distinct"></a>approx_count_distinct

approx_count_distinct(expr[, relativeSD]) - 返回 HyperLogLog++ 估算的基数。 `relativeSD` 定义允许的最大估算错误。

## <a name="approx_percentile"></a>approx_percentile

approx_percentile(col, percentage [, accuracy]) - 按给定百分比返回数值列 `col` 的近似百分位值。 百分比的值必须介于 0.0 到 1.0 之间。 `accuracy` 参数（默认值：10000）是一个正值文本，它以内存为代价控制近似精度。 `accuracy` 的值越高，精度越高。`1.0/accuracy` 是近似计算的相对错误。 当 `percentage` 为数组时，百分比数组的每个值必须介于 0.0 到 1.0 之间。 在这种情况下，会按给定百分比数组返回列 `col` 的大致百分位数组。

**示例：**

```sql
> SELECT approx_percentile(10.0, array(0.5, 0.4, 0.1), 100);
 [10.0,10.0,10.0]
> SELECT approx_percentile(10.0, 0.5, 100);
 10.0
```

## <a name="array"></a>array

array(expr, …) - 返回一个包含给定元素的数组。

**示例：**

```sql
> SELECT array(1, 2, 3);
 [1,2,3]
```

## <a name="array_contains"></a>array_contains

array_contains(array, value) - 如果数组包含该值，则返回 true。

**示例：**

```sql
> SELECT array_contains(array(1, 2, 3), 2);
 true
```

## <a name="array_distinct"></a>array_distinct

array_distinct(array) - 从数组中删除重复值。

**示例：**

```sql
> SELECT array_distinct(array(1, 2, 3, null, 3));
 [1,2,3,null]
```

**最低生效版本：** 2.4.0

## <a name="array_except"></a>array_except

array_except(array1, array2) - 返回存在于 array1 中但不存在于 array2 中的元素的数组，不包含重复项。

**示例：**

```sql
> SELECT array_except(array(1, 2, 3), array(1, 3, 5));
 [2]
```

**最低生效版本：** 2.4.0

## <a name="array_intersect"></a>array_intersect

array_intersect(array1, array2) - 返回 array1 和 array2 的交集中的元素的数组，不包含重复项。

**示例：**

```sql
> SELECT array_intersect(array(1, 2, 3), array(1, 3, 5));
 [1,3]
```

**最低生效版本：** 2.4.0

## <a name="array_join"></a>array_join

array_join(array, delimiter[, nullReplacement]) - 使用分隔符连接给定数组的元素，并使用可选字符串替换数组中的 null 值。 如果没有为 nullReplacement 设置任何值，则会筛选掉任何 null 值。

**示例：**

```sql
> SELECT array_join(array('hello', 'world'), ' ');
 hello world
> SELECT array_join(array('hello', null ,'world'), ' ');
 hello world
> SELECT array_join(array('hello', null ,'world'), ' ', ',');
 hello , world
```

**最低生效版本：** 2.4.0

## <a name="array_max"></a>array_max

array_max(array) - 返回数组中的最大值。 将跳过 NULL 元素。

**示例：**

```sql
> SELECT array_max(array(1, 20, null, 3));
 20
```

**最低生效版本：** 2.4.0

## <a name="array_min"></a>array_min

array_min(array) - 返回数组中的最小值。 将跳过 NULL 元素。

**示例：**

```sql
> SELECT array_min(array(1, 20, null, 3));
 1
```

**最低生效版本：** 2.4.0

## <a name="array_position"></a>array_position

array_position(array, element) - 以长整型返回数组的第一个元素的索引（从 1 开始）。

**示例：**

```sql
> SELECT array_position(array(3, 2, 1), 1);
 3
```

**最低生效版本：** 2.4.0

## <a name="array_remove"></a>array_remove

array_remove(array, element) - 删除数组中与 element 相等的所有元素。

**示例：**

```sql
> SELECT array_remove(array(1, 2, 3, null, 3), 3);
 [1,2,null]
```

**最低生效版本：** 2.4.0

## <a name="array_repeat"></a>array_repeat

array_repeat(element, count) - 返回一个数组，其中包含的 element 重复 count 次。

**示例：**

```sql
> SELECT array_repeat('123', 2);
 ["123","123"]
```

**最低生效版本：** 2.4.0

## <a name="array_sort"></a>array_sort

array_sort(array) - 按升序对输入数组排序。 输入数组的元素必须可排序。 将在返回的数组末尾放置 Null 元素。

**示例：**

```sql
> SELECT array_sort(array('b', 'd', null, 'c', 'a'));
 ["a","b","c","d",null]
```

**最低生效版本：** 2.4.0

## <a name="array_union"></a>array_union

array_union(array1, array2) - 返回 array1 和 array2 的并集中的元素的数组，不包含重复项。

**示例：**

```sql
> SELECT array_union(array(1, 2, 3), array(1, 3, 5));
 [1,2,3,5]
```

**最低生效版本：** 2.4.0

## <a name="arrays_overlap"></a>arrays_overlap

arrays_overlap(a1, a2) - 如果 a1 至少包含一个也存在于 a2 中的非 null 元素，则返回 true。 如果数组没有公共元素，均为非空数组，并且其中任何一个包含 null 元素，则返回 null 元素，否则返回 false。

**示例：**

```sql
> SELECT arrays_overlap(array(1, 2, 3), array(3, 4, 5));
 true
```

**最低生效版本：** 2.4.0

## <a name="arrays_zip"></a>arrays_zip

arrays_zip(a1, a2, …) - 返回合并的结构数组，其中的第 N 个结构包含输入数组的所有的第 N 个值。

**示例：**

```sql
> SELECT arrays_zip(array(1, 2, 3), array(2, 3, 4));
```

```
 [{"0":1,"1":2},{"0":2,"1":3},{"0":3,"1":4}]
```

```sql
> SELECT arrays_zip(array(1, 2), array(2, 3), array(3, 4));
```

```
 [{"0":1,"1":2,"2":3},{"0":2,"1":3,"2":4}]
```

**最低生效版本：** 2.4.0

## <a name="ascii"></a>ascii

ascii(str) - 返回 `str` 的第一个字符的数值。

**示例：**

```sql
> SELECT ascii('222');
 50
> SELECT ascii(2);
 50
```

## <a name="asin"></a>asin

asin(expr) - 返回 `expr` 的反正弦，就像通过 `java.lang.Math.asin` 计算得出的一样。

**示例：**

```sql
> SELECT asin(0);
 0.0
> SELECT asin(2);
 NaN
```

## <a name="assert_true"></a>assert_true

assert_true(expr) - 如果 `expr` 不为 true，则会引发异常。

**示例：**

```sql
> SELECT assert_true(0 < 1);
 NULL
```

## <a name="atan"></a>atan

atan(expr) - 返回 `expr` 的反正切，就像通过 `java.lang.Math.atan` 计算得出的一样。

**示例：**

```sql
> SELECT atan(0);
 0.0
```

## <a name="atan2"></a>atan2

atan2(exprY, exprX) - 返回平面正 X 轴与坐标 (`exprX`, `exprY`) 所代表的点之间的弧角，就像通过 `java.lang.Math.atan2` 计算得出的一样。

**参数：**

* exprY - y 轴坐标
* exprX - x 轴坐标

**示例：**

```sql
> SELECT atan2(0, 0);
 0.0
```

## <a name="avg"></a>平均值

avg(expr) - 返回从组的值计算出的平均值。

## <a name="base64"></a>base64

base64(bin) - 将参数从二进制 `bin` 转换为 base64 字符串。

**示例：**

```sql
> SELECT base64('Spark SQL');
 U3BhcmsgU1FM
```

## <a name="bigint"></a>bigint

bigint(expr) - 将值 `expr` 强制转换为目标数据类型 `bigint`。

## <a name="bin"></a>bin

bin(expr) - 返回以二进制表示的长整型值 `expr` 的字符串表示形式。

**示例：**

```sql
> SELECT bin(13);
 1101
> SELECT bin(-13);
 1111111111111111111111111111111111111111111111111111111111110011
> SELECT bin(13.3);
 1101
```

## <a name="binary"></a>binary

binary(expr) - 将值 `expr` 强制转换为目标数据类型 `binary`。

## <a name="bit_length"></a>bit_length

bit_length(expr) - 返回字符串数据的位长度或二进制数据的位数。

**示例：**

```sql
> SELECT bit_length('Spark SQL');
 72
```

## <a name="boolean"></a>boolean

boolean(expr) - 将值 `expr` 强制转换为目标数据类型 `boolean`。

## <a name="bround"></a>bround

bround(expr, d) - 返回已使用 HALF_EVEN 舍入模式舍入为 `d` 个小数位的 `expr`。

**示例：**

```sql
> SELECT bround(2.5, 0);
 2.0
```

## <a name="cardinality"></a>基数

cardinality(expr) - 返回数组或映射的大小。 如果函数的输入为 null，并且 spark.sql.legacy.sizeOfNull 设置为 true，则该函数返回 -1。 如果 spark.sql.legacy.sizeOfNull 设置为 false，则函数会针对 null 输入返回 null。 默认情况下，spark.sql.legacy.sizeOfNull 参数设置为 true。

**示例：**

```sql
> SELECT cardinality(array('b', 'd', 'c', 'a'));
 4
> SELECT cardinality(map('a', 1, 'b', 2));
 2
> SELECT cardinality(NULL);
 -1
```

## <a name="cast"></a>强制转换

cast(expr AS type) - 将值 `expr` 强制转换为目标数据类型 `type`。

**示例：**

```sql
> SELECT cast('10' as int);
 10
```

## <a name="cbrt"></a>cbrt

cbrt(expr) - 返回 `expr` 的立方根。

**示例：**

```sql
> SELECT cbrt(27.0);
 3.0
```

## <a name="ceil"></a>ceil

ceil(expr) - 返回不小于 `expr` 的最小整数。

**示例：**

```sql
> SELECT ceil(-0.1);
 0
> SELECT ceil(5);
 5
```

## <a name="ceiling"></a>ceiling

ceiling(expr) - 返回不小于 `expr` 的最小整数。

**示例：**

```sql
> SELECT ceiling(-0.1);
 0
> SELECT ceiling(5);
 5
```

## <a name="char"></a>char

char(expr) - 返回其二进制形式等效于 `expr` 的 ASCII 字符。 如果 n 大于 256，则结果等效于 chr(n %
256)

**示例：**

```sql
> SELECT char(65);
 A
```

## <a name="char_length"></a>char_length

char_length(expr) - 返回字符串数据的字符长度或二进制数据的字节数。 字符串数据的长度包括尾随空格的长度。 二进制数据的长度包括二进制零的长度。

**示例：**

```sql
> SELECT char_length('Spark SQL ');
 10
> SELECT CHAR_LENGTH('Spark SQL ');
 10
> SELECT CHARACTER_LENGTH('Spark SQL ');
 10
```

## <a name="character_length"></a>character_length

character_length(expr) - 返回字符串数据的字符长度或二进制数据的字节数。 字符串数据的长度包括尾随空格的长度。 二进制数据的长度包括二进制零的长度。

**示例：**

```sql
> SELECT character_length('Spark SQL ');
 10
> SELECT CHAR_LENGTH('Spark SQL ');
 10
> SELECT CHARACTER_LENGTH('Spark SQL ');
 10
```

## <a name="chr"></a>chr

chr(expr) - 返回其二进制形式等效于 `expr` 的 ASCII 字符。 如果 n 大于 256，则结果等效于 chr(n %
256)

**示例：**

```sql
> SELECT chr(65);
 A
```

## <a name="coalesce"></a>coalesce

coalesce(expr1, expr2, …) - 返回第一个非 null 参数（如果存在）。 否则为 null。

**示例：**

```sql
> SELECT coalesce(NULL, 1, NULL);
 1
```

## <a name="collect_list"></a>collect_list

collect_list(expr) - 收集并返回非唯一元素的列表。

## <a name="collect_set"></a>collect_set

collect_set(expr) - 收集并返回一组非重复元素。

## <a name="concat"></a>concat

concat(col1, col2, …, colN) - 返回将 col1、col2、...、colN 串联后的结果。

**示例：**

```sql
  > SELECT concat('Spark', 'SQL');
   SparkSQL
  > SELECT concat(array(1, 2, 3), array(4, 5), array(6));
   [1,2,3,4,5,6]
at logic for arrays is available since 2.4.0.
```

## <a name="concat_ws"></a>concat_ws

concat_ws(sep, [str | array(str)]+) - 返回将以 `sep` 分隔的字符串户进行串联后的结果。

**示例：**

```sql
> SELECT concat_ws(' ', 'Spark', 'SQL');
  Spark SQL
```

## <a name="conv"></a>conv

conv(num, from_base, to_base) - 将 `num` 从 `from_base` 转换为 `to_base`。

**示例：**

```sql
> SELECT conv('100', 2, 10);
 4
> SELECT conv(-10, 16, -10);
 -16
```

## <a name="corr"></a>相关性

corr(expr1, expr2) - 返回表示一组数字对之间的关联情况的皮尔逊系数。

## <a name="cos"></a>cos

cos(expr) - 返回 `expr` 的余弦，就像通过 `java.lang.Math.cos` 计算得出的一样。

**参数：**

* expr - 以弧度表示的角度

**示例：**

```sql
> SELECT cos(0);
 1.0
```

## <a name="cosh"></a>cosh

cosh(expr) - 返回 `expr` 的双曲余弦，就像通过 `java.lang.Math.cosh` 计算得出的一样。

**参数：**

* expr - 双曲角度

**示例：**

```sql
> SELECT cosh(0);
 1.0
```

## <a name="cot"></a>cot

cot(expr) - 返回 `expr` 的余切，就像通过 `1/java.lang.Math.cot` 计算得出的一样。

**参数：**

* expr - 以弧度表示的角度

**示例：**

```sql
> SELECT cot(1);
 0.6420926159343306
```

## <a name="count"></a>count

count(*) - 返回检索到的行的总数，包括那些包含 null 的行。

count(expr[, expr…]) - 返回为其提供的表达式均为非 null 值的行的数目。

count(DISTINCT expr[, expr…]) - 返回为其提供的表达式为唯一的非 null 值的行的数目。

## <a name="count_min_sketch"></a>count_min_sketch

count_min_sketch(col, eps, confidence, seed) - 返回具有给定 esp、置信度和种子的列的 Count-min sketch。 结果是一个字节数组，可以在使用之前将其反序列化为 `CountMinSketch`。 Count-min sketch 是一个概率数据结构，用于通过子线性空间进行基数估算。

## <a name="covar_pop"></a>covar_pop

covar_pop(expr1, expr2) - 返回一组数字对的总体协方差。

## <a name="covar_samp"></a>covar_samp

covar_samp(expr1, expr2) - 返回一组数字对的样本协方差。

## <a name="crc32"></a>crc32

crc32(expr) - 以 bigint 形式返回 `expr` 的循环冗余检查值。

**示例：**

```sql
> SELECT crc32('Spark');
 1557323817
```

## <a name="cube"></a>多维数据集

## <a name="cume_dist"></a>cume_dist

cume_dist() - 计算某个值相对于分区中所有值的位置。

## <a name="current_database"></a>current_database

current_database() - 返回当前数据库。

**示例：**

```sql
> SELECT current_database();
 default
```

## <a name="current_date"></a>current_date

current_date() - 返回查询计算开始时的当前日期。

**最低生效版本：** 1.5.0

## <a name="current_timestamp"></a>current_timestamp

current_timestamp() - 返回查询计算开始时的当前时间戳。

**最低生效版本：** 1.5.0

## <a name="date"></a>date

date(expr) - 将值 `expr` 强制转换为目标数据类型 `date`。

## <a name="date_add"></a>date_add

date_add(start_date, num_days) - 返回 `start_date` 之后 `num_days` 天的日期。

**示例：**

```sql
> SELECT date_add('2016-07-30', 1);
 2016-07-31
```

**最低生效版本：** 1.5.0

## <a name="date_format"></a>date_format

date_format(timestamp, fmt) - 将 `timestamp` 转换为一个字符串值，该值采用日期格式 `fmt` 所指定的格式。

**示例：**

```sql
> SELECT date_format('2016-04-08', 'y');
 2016
```

**最低生效版本：** 1.5.0

## <a name="date_sub"></a>date_sub

date_sub(start_date, num_days) - 返回 `start_date` 之前 `num_days` 天的日期。

**示例：**

```sql
> SELECT date_sub('2016-07-30', 1);
 2016-07-29
```

**最低生效版本：** 1.5.0

## <a name="date_trunc"></a>date_trunc

date_trunc(fmt, ts) - 返回已截断到由格式模型 `fmt` 指定的单位的时间戳 `ts`。 `fmt` 应该是 [“YEAR”, “YYYY”, “YY”, “MON”, “MONTH”, “MM”, “DAY”, “DD”, “HOUR”, “MINUTE”, “SECOND”, “WEEK”, “QUARTER”] 中的一个

**示例：**

```sql
> SELECT date_trunc('YEAR', '2015-03-05T09:32:05.359');
 2015-01-01 00:00:00
> SELECT date_trunc('MM', '2015-03-05T09:32:05.359');
 2015-03-01 00:00:00
> SELECT date_trunc('DD', '2015-03-05T09:32:05.359');
 2015-03-05 00:00:00
> SELECT date_trunc('HOUR', '2015-03-05T09:32:05.359');
 2015-03-05 09:00:00
```

**最低生效版本：** 2.3.0

## <a name="datediff"></a>datediff

datediff(endDate, startDate) - 返回从 `startDate` 到 `endDate` 的天数。

**示例：**

```sql
> SELECT datediff('2009-07-31', '2009-07-30');
 1

> SELECT datediff('2009-07-30', '2009-07-31');
 -1
```

**最低生效版本：** 1.5.0

## <a name="day"></a>day

day(date) - 返回日期/时间戳的月份日期。

**示例：**

```sql
> SELECT day('2009-07-30');
 30
```

**最低生效版本：** 1.5.0

## <a name="dayofmonth"></a>dayofmonth

dayofmonth(date) - 返回日期/时间戳的月份日期。

**示例：**

```sql
> SELECT dayofmonth('2009-07-30');
 30
```

**最低生效版本：** 1.5.0

## <a name="dayofweek"></a>dayofweek

dayofweek(date) - 返回日期/时间戳的星期日期（1 = 星期日，2 = 星期一，...，7 = 星期六）。

**示例：**

```sql
> SELECT dayofweek('2009-07-30');
 5
```

**最低生效版本：** 2.3.0

## <a name="dayofyear"></a>dayofyear

dayofyear(date) - 返回日期/时间戳的年份日期。

**示例：**

```sql
> SELECT dayofyear('2016-04-09');
 100
```

**最低生效版本：** 1.5.0

## <a name="decimal"></a>Decimal

decimal(expr) - 将值 `expr` 强制转换为目标数据类型 `decimal`。

## <a name="decode"></a>decode

decode(bin, charset) - 使用第二个参数字符集对第一个参数进行解码。

**示例：**

```sql
> SELECT decode(encode('abc', 'utf-8'), 'utf-8');
 abc
```

## <a name="degrees"></a>度

degrees(expr) - 将弧度转换为度。

**参数：**

* expr - 以弧度表示的角度

**示例：**

```sql
> SELECT degrees(3.141592653589793);
 180.0
```

## <a name="dense_rank"></a>dense_rank

dense_rank() - 计算某个值在一组值中的排名。 结果是 1 加上之前分配的排名值。 与函数排名不同，dense_rank 不会在排名序列中产生空隙。

## <a name="double"></a>double

double(expr) - 将值 `expr` 强制转换为目标数据类型 `double`。

## <a name="e"></a>E

e() - 返回欧拉数 e。

**示例：**

```sql
> SELECT e();
 2.718281828459045
```

## <a name="element_at"></a>element_at

element_at(array, index) - 返回数组中给定索引（从 1 开始）处的元素。 如果索引 < 0，则以倒序方式访问元素。
如果索引超出数组的长度，则返回 NULL。

element_at(map, key) - 返回给定键的值，如果映射中未包含该键，则返回 NULL

**示例：**

```sql
> SELECT element_at(array(1, 2, 3), 2);
 2
> SELECT element_at(map(1, 'a', 2, 'b'), 2);
 b
```

**最低生效版本：** 2.4.0

## <a name="elt"></a>elt

elt(n, input1, input2, …) - 返回第 `n` 个输入，例如，当 `n` 为 2 时返回 `input2`。

**示例：**

```sql
> SELECT elt(1, 'scala', 'java');
 scala
```

## <a name="encode"></a>encode

encode(str, charset) - 使用第二个参数字符集对第一个参数进行编码。

**示例：**

```sql
> SELECT encode('abc', 'utf-8');
 abc
```

## <a name="exists"></a>exists

exists(expr, pred) - 测试谓词对数组中的一个或多个元素是否成立。

**示例：**

```sql
> SELECT exists(array(1, 2, 3), x -> x % 2 == 0);
 true
```

**最低生效版本：** 2.4.0

## <a name="exp"></a>exp

exp(expr) - 返回 e 的 `expr` 次幂。

**示例：**

```sql
> SELECT exp(0);
 1.0
```

## <a name="explode"></a>explode

explode(expr) - 将数组 `expr` 的元素分为多个行，或将映射 `expr` 的元素分为多个行和列。

**示例：**

```sql
> SELECT explode(array(10, 20));
 10
 20
```

## <a name="explode_outer"></a>explode_outer

explode_outer(expr) - 将数组 `expr` 的元素分为多个行，或将映射 `expr` 的元素分为多个行和列。

**示例：**

```sql
> SELECT explode_outer(array(10, 20));
 10
 20
```

## <a name="expm1"></a>expm1

expm1(expr) - 返回 exp(`expr`) - 1。

**示例：**

```sql
> SELECT expm1(0);
 0.0
```

## <a name="factorial"></a>阶乘

factorial(expr) - 返回 `expr` 的阶乘。 `expr` 为 [0..20]。 否则为 null。

**示例：**

```sql
> SELECT factorial(5);
 120
```

## <a name="filter"></a>filter

filter(expr, func) - 使用给定的谓词筛选输入数组。

**示例：**

```sql
> SELECT filter(array(1, 2, 3), x -> x % 2 == 1);
 [1,3]
```

**最低生效版本：** 2.4.0

## <a name="find_in_set"></a>find_in_set

find_in_set(str, str_array) - 返回以逗号分隔的列表 (`str_array`) 中的给定字符串 (`str`) 的索引（从 1 开始）。 如果未找到该字符串或者给定字符串 (`str`) 包含逗号，则返回 0。

**示例：**

```sql
> SELECT find_in_set('ab','abc,b,ab,c,def');
 3
```

## <a name="first"></a>first

first(expr[, isIgnoreNull]) - 为一组行返回 `expr` 的第一个值。 如果 `isIgnoreNull` 为 true，则仅返回非 null 值。

## <a name="first_value"></a>first_value

first_value(expr[, isIgnoreNull]) - 为一组行返回 `expr` 的第一个值。 如果 `isIgnoreNull` 为 true，则仅返回非 null 值。

## <a name="flatten"></a>平展 (flatten)

flatten(arrayOfArrays) - 将数组的数组转换为单个数组。

**示例：**

```sql
> SELECT flatten(array(array(1, 2), array(3, 4)));
 [1,2,3,4]
```

**最低生效版本：** 2.4.0

## <a name="float"></a>FLOAT

float(expr) - 将值 `expr` 强制转换为目标数据类型 `float`。

## <a name="floor"></a>floor

floor(expr) - 返回不大于 `expr` 的最大整数。

**示例：**

```sql
> SELECT floor(-0.1);
 -1
> SELECT floor(5);
 5
```

## <a name="format_number"></a>format_number

format_number(expr1, expr2) - 设置“#,###,###.##”之类的数字 `expr1` 的格式，将其舍入到 `expr2` 个小数位。 如果 `expr2` 为 0，则结果不包含小数点或小数部分。 `expr2` 还接受用户指定的格式。 此函数的作用应该类似于 MySQL 的 FORMAT。

**示例：**

```sql
> SELECT format_number(12332.123456, 4);
 12,332.1235
> SELECT format_number(12332.123456, '##################.###');
 12332.123
```

## <a name="format_string"></a>format_string

format_string(strfmt, obj, …) - 从 printf 样式的格式字符串返回一个带格式的字符串。

**示例：**

```sql
> SELECT format_string("Hello World %d %s", 100, "days");
 Hello World 100 days
```

## <a name="from_json"></a>from_json

from_json(jsonStr, schema[, options]) - 返回具有给定 `jsonStr` 和 `schema` 的结构值。

**示例：**

```sql
> SELECT from_json('{"a":1, "b":0.8}', 'a INT, b DOUBLE');
```

```
 {"a":1, "b":0.8}
```

```sql
> SELECT from_json('{"time":"26/08/2015"}', 'time Timestamp', map('timestampFormat', 'dd/MM/yyyy'));
```

```
 {"time":"2015-08-26 00:00:00.0"}
```

**最低生效版本：** 2.2.0

## <a name="from_unixtime"></a>from_unixtime

from_unixtime(unix_time, format) - 返回采用指定 `format` 的 `unix_time`。

**示例：**

```sql
> SELECT from_unixtime(0, 'yyyy-MM-dd HH:mm:ss');
 1970-01-01 00:00:00
```

**最低生效版本：** 1.5.0

## <a name="from_utc_timestamp"></a>from_utc_timestamp

from_utc_timestamp(timestamp, timezone) - 在给定时间戳（例如“2017-07-14 02:40:00.0”）的情况下，将其解释为 UTC 时间，并将该时间呈现为给定时区中的时间戳。 例如，“GMT+1”会生成“2017-07-14 03:40:00.0”。

**示例：**

```sql
> SELECT from_utc_timestamp('2016-08-31', 'Asia/Seoul');
 2016-08-31 09:00:00
```

**最低生效版本：** 1.5.0

## <a name="get_json_object"></a>get_json_object

get_json_object(json_txt, path) - 从 `path` 中提取 json 对象。

**示例：**

```sql
> SELECT get_json_object('{"a":"b"}', '$.a');
 b
```

## <a name="greatest"></a>greatest

greatest(expr, …) - 返回所有参数的最大值（跳过 null 值）。

**示例：**

```sql
> SELECT greatest(10, 9, 2, 4, 3);
 10
```

## <a name="grouping"></a>分组

## <a name="grouping_id"></a>grouping_id

## <a name="hash"></a>hash

hash(expr1, expr2, …) - 返回参数的哈希值。

**示例：**

```sql
> SELECT hash('Spark', array(123), 2);
 -1321691492
```

## <a name="hex"></a>hex

hex(expr) - 将 `expr` 转换为十六进制。

**示例：**

```sql
> SELECT hex(17);
 11
> SELECT hex('Spark SQL');
 537061726B2053514C
```

## <a name="hour"></a>hour

hour(timestamp) - 返回字符串/时间戳的小时部分。

**示例：**

```sql
> SELECT hour('2009-07-30 12:58:59');
 12
```

**最低生效版本：** 1.5.0

## <a name="hypot"></a>hypot

hypot(expr1, expr2) - 返回 sqrt(`expr1`**2 + `expr2`** 2)。

**示例：**

```sql
> SELECT hypot(3, 4);
 5.0
```

## <a name="if"></a>if

if(expr1, expr2, expr3) - 如果 `expr1` 的计算结果为 true，则返回 `expr2`，否则返回 `expr3`。

**示例：**

```sql
> SELECT if(1 < 2, 'a', 'b');
 a
```

## <a name="ifnull"></a>ifnull

ifnull(expr1, expr2) - 如果 `expr1` 为 null，则返回 `expr2`，否则返回 `expr1`。

**示例：**

```sql
> SELECT ifnull(NULL, array('2'));
 ["2"]
```

## <a name="in"></a>in

expr1 in(expr2, expr3, …) - 如果 `expr` 等于任何 valN，则返回 true。

**参数：**

* expr1, expr2, expr3, … - 这些参数必须是同一类型。

**示例：**

```sql
> SELECT 1 in(1, 2, 3);
 true
> SELECT 1 in(2, 3, 4);
 false
> SELECT named_struct('a', 1, 'b', 2) in(named_struct('a', 1, 'b', 1), named_struct('a', 1, 'b', 3));
 false
> SELECT named_struct('a', 1, 'b', 2) in(named_struct('a', 1, 'b', 2), named_struct('a', 1, 'b', 3));
 true
```

## <a name="initcap"></a>initcap

initcap(str) - 返回 `str`，其中的每个单词的首字母大写。 所有其他字母都小写。 单词由空格分隔。

**示例：**

```sql
> SELECT initcap('sPark sql');
 Spark Sql
```

## <a name="inline"></a>inline

inline(expr) - 将结构数组分解为一个表。

**示例：**

```sql
> SELECT inline(array(struct(1, 'a'), struct(2, 'b')));
 1  a
 2  b
```

## <a name="inline_outer"></a>inline_outer

inline_outer(expr) - 将结构数组分解为一个表。

**示例：**

```sql
> SELECT inline_outer(array(struct(1, 'a'), struct(2, 'b')));
 1  a
 2  b
```

## <a name="input_file_block_length"></a>input_file_block_length

input_file_block_length() - 返回正在读取的块的长度。如果无法获得该长度，则返回 -1。

## <a name="input_file_block_start"></a>input_file_block_start

input_file_block_start() - 返回正在读取的块的起始偏移量。如果无法获得该偏移量，则返回 -1。

## <a name="input_file_name"></a>input_file_name

input_file_name() - 返回正在读取的文件的名称。如果无法获得该名称，则返回空字符串。

## <a name="instr"></a>instr

instr(str, substr) - 返回 `str` 中 `substr` 的第一个匹配项的索引（从 1 开始）。

**示例：**

```sql
> SELECT instr('SparkSQL', 'SQL');
 6
```

## <a name="int"></a>int

int(expr) - 将值 `expr` 强制转换为目标数据类型 `int`。

## <a name="isnan"></a>isnan

isnan(expr) - 如果 `expr` 为 NAN，则返回 true，否则返回 false。

**示例：**

```sql
> SELECT isnan(cast('NaN' as double));
 true
```

## <a name="isnotnull"></a>isnotnull

isnotnull(expr) - 如果 `expr` 不为 null，则返回 true，否则返回 false。

**示例：**

```sql
> SELECT isnotnull(1);
 true
```

## <a name="isnull"></a>isnull

isnull(expr) - 如果 `expr` 为 null，则返回 true，否则返回 false。

**示例：**

```sql
> SELECT isnull(1);
 false
```

## <a name="java_method"></a>java_method

java_method(class, method[, arg1[, arg2 ..]]) - 使用反射调用某个方法。

**示例：**

```sql
> SELECT java_method('java.util.UUID', 'randomUUID');
 c33fb387-8500-4bfa-81d2-6e0e3e930df2
> SELECT java_method('java.util.UUID', 'fromString', 'a5cf6c42-0c85-418f-af6c-3e4e5b1328f2');
 a5cf6c42-0c85-418f-af6c-3e4e5b1328f2
```

## <a name="json_tuple"></a>json_tuple

json_tuple(jsonStr, p1, p2, …, pn) - 像函数 get_json_object 一样返回一个元组，但采用多个名称。 所有输入参数和输出列类型均为字符串。

**示例：**

```sql
> SELECT json_tuple('{"a":1, "b":2}', 'a', 'b');
 1  2
```

## <a name="kurtosis"></a>kurtosis

kurtosis(expr) - 返回从组的值计算出的峰度值。

## <a name="lag"></a>lag

lag(input[, offset[, default]]) - 返回窗口中当前行之前第 `offset` 行的 `input` 的值。 `offset` 的默认值为 1，`default` 的默认值为 null。
如果第 `offset` 行的 `input` 的值为 null，则返回 null。 如果没有此类偏移行（例如，当偏移量为 1 时，窗口的第一行没有任何以前的行），则返回 `default`。

## <a name="last"></a>last

last(expr[, isIgnoreNull]) - 为一组行返回 `expr` 的最后一个值。 如果 `isIgnoreNull` 为 true，则仅返回非 null 值。

## <a name="last_day"></a>last_day

last_day(date) - 返回日期所属月份的最后一天。

**示例：**

```sql
> SELECT last_day('2009-01-12');
 2009-01-31
```

**最低生效版本：** 1.5.0

## <a name="last_value"></a>last_value

last_value(expr[, isIgnoreNull]) - 为一组行返回 `expr` 的最后一个值。 如果 `isIgnoreNull` 为 true，则仅返回非 null 值。

## <a name="lcase"></a>lcase

lcase(str) - 返回 `str`，其中的所有字符均更改为小写。

**示例：**

```sql
> SELECT lcase('SparkSql');
 sparksql
```

## <a name="lead"></a>lead

lead(input[, offset[, default]]) - 返回窗口中当前行之后第 `offset` 行的 `input` 的值。 `offset` 的默认值为 1，`default` 的默认值为 null。
如果第 `offset` 行的 `input` 的值为 null，则返回 null。 如果没有此类偏移行（例如，当偏移量为 1 时，窗口的最后一行没有任何后续行），则返回 `default`。

## <a name="least"></a>least

least(expr, …) - 返回所有参数的最小值（跳过 null 值）。

**示例：**

```sql
> SELECT least(10, 9, 2, 4, 3);
 2
```

## <a name="left"></a>左侧

left(str, len) - 返回字符串 `str` 最左侧的 `len`（`len` 可以为字符串类型）个字符；如果 `len` 小于或等于 0，则结果为空字符串。

**示例：**

```sql
> SELECT left('Spark SQL', 3);
 Spa
```

## <a name="length"></a>length

length(expr) - 返回字符串数据的字符长度或二进制数据的字节数。 字符串数据的长度包括尾随空格的长度。 二进制数据的长度包括二进制零的长度。

**示例：**

```sql
> SELECT length('Spark SQL ');
 10
> SELECT CHAR_LENGTH('Spark SQL ');
 10
> SELECT CHARACTER_LENGTH('Spark SQL ');
 10
```

## <a name="levenshtein"></a>levenshtein

levenshtein(str1, str2) - 返回两个给定字符串之间的 Levenshtein 距离。

**示例：**

```sql
> SELECT levenshtein('kitten', 'sitting');
 3
```

## <a name="like"></a>like

str like pattern - 如果 str 与 pattern 匹配，则返回 true；如果任何参数为 null，则返回 null；在其他情况下返回 false。

**参数：**

* str - 字符串表达式
* pattern - 字符串表达式。 pattern 是在字面上匹配的字符串，但以下特殊符号例外：

  _ 匹配输入中的任一字符（类似于 posix 正则表达式中的 .）

  % 匹配输入中的零个或多个字符（类似于 posix 正则表达式中的 .*）

  转义字符为 ‘’。 如果转义字符的之前带有特殊符号或其他转义字符，则在字面上匹配后面的字符。 转义其他任何字符的操作无效。

  从 Spark 2.0 开始，字符串文本在我们的 SQL 分析器中不转义。 例如，为了与“abc”匹配，pattern 应为“abc”。

  启用了 SQL 配置“spark.sql.parser.escapedStringLiterals”时，它会回退到与字符串文本分析有关的 Spark 1.6 行为。
  例如，如果启用了此配置，则与“abc”匹配的 pattern 应为“abc”。

**示例：**

```sql
> SELECT '%SystemDrive%\Users\John' like '\%SystemDrive\%\\Users%'
true
```

**注意：**

使用 RLIKE 与标准正则表达式匹配。

## <a name="ln"></a>ln

ln(expr) - 返回 `expr` 的自然对数（以 e 为底）。

**示例：**

```sql
> SELECT ln(1);
 0.0
```

## <a name="locate"></a>locate

locate(substr, str[, pos]) - 返回 `substr` 第一次出现在 `str` 中位置 `pos` 之后的位置。 给定的 `pos` 和返回值是从 1 开始的。

**示例：**

```sql
> SELECT locate('bar', 'foobarbar');
 4
> SELECT locate('bar', 'foobarbar', 5);
 7
> SELECT POSITION('bar' IN 'foobarbar');
 4
```

## <a name="log"></a>log

log(base, expr) - 返回 `expr` 的对数（以 `base` 为底）。

**示例：**

```sql
> SELECT log(10, 100);
 2.0
```

## <a name="log10"></a>log10

log10(expr) - 返回 `expr` 的对数（以 10 为底）。

**示例：**

```sql
> SELECT log10(10);
 1.0
```

## <a name="log1p"></a>log1p

log1p(expr) - 返回 log(1 + `expr`)。

**示例：**

```sql
> SELECT log1p(0);
 0.0
```

## <a name="log2"></a>log2

log2(expr) - 返回 `expr` 的对数（以 2 为底）。

**示例：**

```sql
> SELECT log2(2);
 1.0
```

## <a name="lower"></a>lower

lower(str) - 返回 `str`，其中所有字符均更改为小写。

**示例：**

```sql
> SELECT lower('SparkSql');
 sparksql
```

## <a name="lpad"></a>lpad

lpad(str, len, pad) - 返回左侧填充了 `pad` 的 `str`，填充后整个字符的长度为 `len`。 如果 `str` 的长度超过 `len`，则返回值将缩短为 `len` 个字符。

**示例：**

```sql
> SELECT lpad('hi', 5, '??');
 ???hi
> SELECT lpad('hi', 1, '??');
 h
```

## <a name="ltrim"></a>ltrim

ltrim(str) - 从 `str` 中删除前导空格字符。

ltrim(trimStr, str) - 删除包含来自剪裁字符串的字符的前导字符串

**参数：**

* str - 字符串表达式
* trimStr - 要剪裁的剪裁字符串字符，默认值为单个空格

**示例：**

```sql
> SELECT ltrim('    SparkSQL   ');
 SparkSQL
> SELECT ltrim('Sp', 'SSparkSQLS');
 arkSQLS
```

## <a name="map"></a>map

map(key0, value0, key1, value1, …) - 创建具有给定键/值对的映射。

**示例：**

```sql
> SELECT map(1.0, '2', 3.0, '4');
```

```
 {1.0:"2",3.0:"4"}
```

## <a name="map_concat"></a>map_concat

map_concat(map, …) - 返回所有给定映射的并集

**示例：**

```sql
> SELECT map_concat(map(1, 'a', 2, 'b'), map(2, 'c', 3, 'd'));
```

```
 {1:"a",2:"c",3:"d"}
```

**最低生效版本：** 2.4.0

## <a name="map_from_arrays"></a>map_from_arrays

map_from_arrays(keys, values) - 创建具有一对给定的键/值数组的映射。 键中的所有元素都不应为 null

**示例：**

```sql
> SELECT map_from_arrays(array(1.0, 3.0), array('2', '4'));
```

```
 {1.0:"2",3.0:"4"}
```

**最低生效版本：** 2.4.0

## <a name="map_from_entries"></a>map_from_entries

map_from_entries(arrayOfEntries) - 返回从给定的条目数组创建的映射。

**示例：**

```sql
> SELECT map_from_entries(array(struct(1, 'a'), struct(2, 'b')));
```

```
 {1:"a",2:"b"}
```

**最低生效版本：** 2.4.0

## <a name="map_keys"></a>map_keys

map_keys(map) - 返回包含映射键的无序数组。

**示例：**

```sql
> SELECT map_keys(map(1, 'a', 2, 'b'));
 [1,2]
```

## <a name="map_values"></a>map_values

map_values(map) - 返回包含映射值的无序数组。

**示例：**

```sql
> SELECT map_values(map(1, 'a', 2, 'b'));
 ["a","b"]
```

## <a name="max"></a>max

max(expr) - 返回 `expr` 的最大值。

## <a name="md5"></a>md5

md5(expr) - 以 `expr` 的十六进制字符串形式返回 MD5 128 位校验和。

**示例：**

```sql
> SELECT md5('Spark');
 8cde774d6f7333752ed72cacddb05126
```

## <a name="mean"></a>平均值

mean(expr) - 返回从组的值计算出的平均值。

## <a name="min"></a>分钟

min(expr) - 返回 `expr` 的最小值。

## <a name="minute"></a>minute

minute(timestamp) - 返回字符串/时间戳的分钟部分。

**示例：**

```sql
> SELECT minute('2009-07-30 12:58:59');
 58
```

**最低生效版本：** 1.5.0

## <a name="mod"></a>mod

expr1 mod expr2 - 返回在执行 `expr1`/`expr2` 后的余数。

**示例：**

```sql
> SELECT 2 mod 1.8;
 0.2
> SELECT MOD(2, 1.8);
 0.2
```

## <a name="monotonically_increasing_id"></a>monotonically_increasing_id

monotonically_increasing_id() - 返回单调递增的 64 位整数。 生成的 ID 保证是单调递增且唯一的，但不是连续的。 当前的实现会将分区 ID 放在较高的 31 位中，而较低的 33 位表示每个分区中的记录号。 当前的实现以下面的假设为前提：数据帧的分区少于 10 亿个，每个分区的记录少于 80 亿个。 此函数为非确定性函数，因为其结果取决于分区 ID。

## <a name="month"></a>月份

month(date) - 返回日期/时间戳的月份部分。

**示例：**

```sql
> SELECT month('2016-07-30');
 7
```

**最低生效版本：** 1.5.0

## <a name="months_between"></a>months_between

months_between(timestamp1, timestamp2[, roundOff]) - 如果 `timestamp1` 晚于 `timestamp2`，则结果为正。 如果 `timestamp1` 和 `timestamp2` 是同一月的同一天，或者两者都是同一月的最后一天，则会忽略当天的时间。 否则，会根据每月 31 天计算差异并将其舍入到 8 位数，除非 roundOff = false。

**示例：**

```sql
> SELECT months_between('1997-02-28 10:30:00', '1996-10-30');
 3.94959677
> SELECT months_between('1997-02-28 10:30:00', '1996-10-30', false);
 3.9495967741935485
```

**最低生效版本：** 1.5.0

## <a name="named_struct"></a>named_struct

named_struct(name1, val1, name2, val2, …) - 创建具有给定字段名称和值的结构。

**示例：**

```sql
> SELECT named_struct("a", 1, "b", 2, "c", 3);
```

```
 {"a":1,"b":2,"c":3}
```

## <a name="nanvl"></a>nanvl

nanvl(expr1, expr2) - 如果 `expr1` 不是 NAN，则返回 expr1，否则返回 `expr2`。

**示例：**

```sql
> SELECT nanvl(cast('NaN' as double), 123);
 123.0
```

## <a name="negative"></a>消极

negative(expr) - 返回 `expr` 的求反值。

**示例：**

```sql
> SELECT negative(1);
 -1
```

## <a name="next_day"></a>next_day

next_day(start_date, day_of_week) - 返回晚于 `start_date` 并已按指示命名的第一个日期。

**示例：**

```sql
> SELECT next_day('2015-01-14', 'TU');
 2015-01-20
```

**最低生效版本：** 1.5.0

## <a name="not"></a>not

not expr - 逻辑非。

## <a name="now"></a>now

now() - 返回查询计算开始时的当前时间戳。

**最低生效版本：** 1.5.0

## <a name="ntile"></a>ntile

ntile(n) - 将每个窗口分区的行分割为从 1 到至多 `n` 的 `n` 个 Bucket。

## <a name="nullif"></a>nullif

nullif(expr1, expr2) - 如果 `expr1` 等于 `expr2`，则返回 null，否则返回 `expr1`。

**示例：**

```sql
> SELECT nullif(2, 2);
 NULL
```

## <a name="nvl"></a>nvl

nvl(expr1, expr2) - 如果 `expr1` 为 null，则返回 `expr2`，否则返回 `expr1`。

**示例：**

```sql
> SELECT nvl(NULL, array('2'));
 ["2"]
```

## <a name="nvl2"></a>nvl2

nvl2(expr1, expr2, expr3) - 如果 `expr1` 不为 null，则返回 `expr2`，否则返回 `expr3`。

**示例：**

```sql
> SELECT nvl2(NULL, 2, 1);
 1
```

## <a name="octet_length"></a>octet_length

octet_length(expr) - 返回字符串数据的字节长度或二进制数据的字节数。

**示例：**

```sql
> SELECT octet_length('Spark SQL');
 9
```

## <a name="or"></a>或

expr1 or expr2 - 逻辑或。

## <a name="parse_url"></a>parse_url

parse_url(url, partToExtract[, key]) - 从 URL 中提取一个部分。

**示例：**

```sql
> SELECT parse_url('https://spark.apache.org/path?query=1', 'HOST')
 spark.apache.org
> SELECT parse_url('https://spark.apache.org/path?query=1', 'QUERY')
 query=1
> SELECT parse_url('https://spark.apache.org/path?query=1', 'QUERY', 'query')
 1
```

## <a name="percent_rank"></a>percent_rank

percent_rank() - 计算某个值在一组值中的百分比排名。

## <a name="percentile"></a>percentile

percentile(col, percentage [, frequency]) - 按给定百分比返回数值列 `col` 的确切百分位值。 百分比的值必须介于 0.0 到 1.0 之间。 频率的值应为正整数

percentile(col, array(percentage1 [, percentage2]…) [, frequency]) - 按给定百分比返回数值列 `col` 的确切百分位值数组。 百分比数组的每个值必须介于 0.0 到 1.0 之间。 频率的值应为正整数

## <a name="percentile_approx"></a>percentile_approx

percentile_approx(col, percentage [, accuracy]) - 按给定百分比返回数值列 `col` 的近似百分位值。 百分比的值必须介于 0.0 到 1.0 之间。 `accuracy` 参数（默认值：10000）是一个正值文本，它以内存为代价控制近似精度。 `accuracy` 的值越高，精度越高。`1.0/accuracy` 是近似计算的相对错误。 当 `percentage` 为数组时，百分比数组的每个值必须介于 0.0 到 1.0 之间。 在这种情况下，会按给定百分比数组返回列 `col` 的大致百分位数组。

**示例：**

```sql
> SELECT percentile_approx(10.0, array(0.5, 0.4, 0.1), 100);
 [10.0,10.0,10.0]
> SELECT percentile_approx(10.0, 0.5, 100);
 10.0
```

## <a name="pi"></a>pi

pi() - 返回 pi。

**示例：**

```sql
> SELECT pi();
 3.141592653589793
```

## <a name="pmod"></a>pmod

pmod(expr1, expr2) - 返回 `expr1` mod `expr2` 的正值。

**示例：**

```sql
> SELECT pmod(10, 3);
 1
> SELECT pmod(-10, 3);
 2
```

## <a name="posexplode"></a>posexplode

posexplode(expr) - 将数组 `expr` 的元素分为多个包含位置的行，或将映射 `expr` 的元素分为多个包含位置的行和列。

**示例：**

```sql
> SELECT posexplode(array(10,20));
 0  10
 1  20
```

## <a name="posexplode_outer"></a>posexplode_outer

posexplode_outer(expr) - 将数组 `expr` 的元素分为多个包含位置的行，或将映射 `expr` 的元素分为多个包含位置的行和列。

**示例：**

```sql
> SELECT posexplode_outer(array(10,20));
 0  10
 1  20
```

## <a name="position"></a>position

position(substr, str[, pos]) - 返回 `substr` 第一次出现在 `str` 中位置 `pos` 之后的位置。 给定的 `pos` 和返回值是从 1 开始的。

**示例：**

```sql
> SELECT position('bar', 'foobarbar');
 4
> SELECT position('bar', 'foobarbar', 5);
 7
> SELECT POSITION('bar' IN 'foobarbar');
 4
```

## <a name="positive"></a>积极

positive(expr) - 返回 `expr` 的值。

## <a name="pow"></a>pow

pow(expr1, expr2) - 求 `expr1` 的 `expr2` 次幂的值。

**示例：**

```sql
> SELECT pow(2, 3);
 8.0
```

## <a name="power"></a>power

power(expr1, expr2) - 求 `expr1` 的 `expr2` 次幂的值。

**示例：**

```sql
> SELECT power(2, 3);
 8.0
```

## <a name="printf"></a>printf

printf(strfmt, obj, …) - 从 printf 样式的格式字符串返回一个带格式的字符串。

**示例：**

```sql
> SELECT printf("Hello World %d %s", 100, "days");
 Hello World 100 days
```

## <a name="quarter"></a>quarter

quarter(date) - 返回日期所对应的年内季度，范围为 1 到 4。

**示例：**

```sql
> SELECT quarter('2016-08-31');
 3
```

**最低生效版本：** 1.5.0

## <a name="radians"></a>radians

radians(expr) - 将度转换为弧度。

**参数：**

* expr - 以度表示的角度

**示例：**

```sql
> SELECT radians(180);
 3.141592653589793
```

## <a name="rand"></a>rand

rand([seed]) - 返回一个随机值，该随机值是 [0, 1) 中具有独立同分布 (i.i.d.) 特性的均匀分布值。

**示例：**

```sql
  > SELECT rand();
   0.9629742951434543
  > SELECT rand(0);
   0.8446490682263027
  > SELECT rand(null);
   0.8446490682263027
function is non-deterministic in general case.
```

## <a name="randn"></a>randn

randn([seed]) - 返回一个随机值，该随机值是从标准正态分布中抽取的具有独立同分布 (i.i.d.) 特性的值。

**示例：**

```sql
  > SELECT randn();
   -0.3254147983080288
  > SELECT randn(0);
   1.1164209726833079
  > SELECT randn(null);
   1.1164209726833079
function is non-deterministic in general case.
```

## <a name="rank"></a>rank

rank() - 计算某个值在值组中的排名。 结果是 1 加上前面的行数，或者等于当前行在分区中的顺序。 值将在序列中生成空隙。

## <a name="reflect"></a>reflect

reflect(class, method[, arg1[, arg2 ..]]) - 使用反射调用某个方法。

**示例：**

```sql
> SELECT reflect('java.util.UUID', 'randomUUID');
 c33fb387-8500-4bfa-81d2-6e0e3e930df2
> SELECT reflect('java.util.UUID', 'fromString', 'a5cf6c42-0c85-418f-af6c-3e4e5b1328f2');
 a5cf6c42-0c85-418f-af6c-3e4e5b1328f2
```

## <a name="regexp_extract"></a>regexp_extract

regexp_extract(str, regexp[, idx]) - 提取与 `regexp` 匹配的组。

**示例：**

```sql
> SELECT regexp_extract('100-200', '(\\d+)-(\\d+)', 1);
 100
```

## <a name="regexp_replace"></a>regexp_replace

regexp_replace(str, regexp, rep) - 将 `str` 中与 `regexp` 匹配的所有子字符串替换为 `rep`。

**示例：**

```sql
> SELECT regexp_replace('100-200', '(\\d+)', 'num');
 num-num
```

## <a name="repeat"></a>repeat

repeat(str, n) - 返回重复给定字符串值 n 次的字符串。

**示例：**

```sql
> SELECT repeat('123', 2);
 123123
```

## <a name="replace"></a>replace

replace(str, search[, replace]) - 将出现的所有 `search` 替换为 `replace`。

**参数：**

* str - 字符串表达式
* search - 字符串表达式。 如果在 `str` 中找不到 `search`，则按原样返回 `str`。
* replace - 字符串表达式。 如果 `replace` 未指定或为空字符串，则不会替换从 `str` 中删除的字符串。

**示例：**

```sql
> SELECT replace('ABCabc', 'abc', 'DEF');
 ABCDEF
```

## <a name="reverse"></a>reverse

reverse(array) - 返回一个反向字符串或一个包含逆序的元素的数组。

**示例：**

```sql
> SELECT reverse('Spark SQL');
 LQS krapS
> SELECT reverse(array(2, 1, 4, 3));
 [3,4,1,2]
```

数组逆序逻辑从 2.4.0 开始提供。 **最低生效版本：** 1.5.0

## <a name="right"></a>右

right(str, len) - 返回字符串 `str` 最右侧的 `len`（`len` 可以为字符串类型）个字符；如果 `len` 小于或等于 0，则结果为空字符串。

**示例：**

```sql
> SELECT right('Spark SQL', 3);
 SQL
```

## <a name="rint"></a>rint

rint(expr) - 返回一个最接近于参数值且等于一个数学整数的双精度值。

**示例：**

```sql
> SELECT rint(12.3456);
 12.0
```

## <a name="rlike"></a>rlike

str rlike regexp - 如果 `str` 与 `regexp` 匹配，则返回 true，否则返回 false。

**参数：**

* str - 字符串表达式
* regexp - 字符串表达式。 模式字符串应为 Java 正则表达式。

  从 Spark 2.0 开始，字符串文本（包括正则表达式模式）在我们的 SQL 分析器中不转义。 例如，若要与“abc”匹配，`regexp` 的正则表达式可以是“^abc$”。

  可以使用 SQL 配置“spark.sql.parser.escapedStringLiterals”回退到与字符串文本分析有关的 Spark 1.6 行为。 例如，如果启用了此配置，则能够与“abc”匹配的 `regexp` 是“^abc$”。

**示例：**

```sql
When spark.sql.parser.escapedStringLiterals is disabled (default).
> SELECT '%SystemDrive%\Users\John' rlike '%SystemDrive%\\Users.*'
true

When spark.sql.parser.escapedStringLiterals is enabled.
> SELECT '%SystemDrive%\Users\John' rlike '%SystemDrive%\Users.*'
true
```

**注意：**

使用 LIKE 可通过简单的字符串模式进行匹配。

## <a name="rollup"></a>rollup

## <a name="round"></a>round

round(expr, d) - 返回已使用 HALF_UP 舍入模式舍入到 `d` 个小数位的 `expr`。

**示例：**

```sql
> SELECT round(2.5, 0);
 3.0
```

## <a name="row_number"></a>row_number

row_number() - 根据窗口分区中的行顺序，为每一行分配唯一的顺序编号（从 1 开始）。

## <a name="rpad"></a>rpad

rpad(str, len, pad) - 返回右侧填充了 `pad` 的 `str`，填充后整个字符的长度为 `len`。 如果 `str` 的长度超过 `len`，则返回值将缩短为 `len` 个字符。

**示例：**

```sql
> SELECT rpad('hi', 5, '??');
 hi???
> SELECT rpad('hi', 1, '??');
 h
```

## <a name="rtrim"></a>rtrim

rtrim(str) - 从 `str` 中删除尾随空格字符。

rtrim(trimStr, str) - 从 `str` 中删除包含来自剪裁字符串的字符的尾随字符串

**参数：**

* str - 字符串表达式
* trimStr - 要剪裁的剪裁字符串字符，默认值为单个空格

**示例：**

```sql
> SELECT rtrim('    SparkSQL   ');
 SparkSQL
> SELECT rtrim('LQSa', 'SSparkSQLS');
 SSpark
```

## <a name="schema_of_json"></a>schema_of_json

schema_of_json(json[, options]) - 以 JSON 字符串的 DDL 格式返回架构。

**示例：**

```sql
> SELECT schema_of_json('[{"col":0}]');
 array<struct<col:int>>
```

**最低生效版本：** 2.4.0

## <a name="second"></a>second

second(timestamp) - 返回字符串/时间戳的秒部分。

**示例：**

```sql
> SELECT second('2009-07-30 12:58:59');
 59
```

**最低生效版本：** 1.5.0

## <a name="sentences"></a>句子

sentences(str[, lang, country]) - 将 `str` 拆分为一个数组，其中包含单词数组。

**示例：**

```sql
> SELECT sentences('Hi there! Good morning.');
 [["Hi","there"],["Good","morning"]]
```

## <a name="sequence"></a>sequence

sequence(start, stop, step) - 生成一个数组，其中包含从 start 到 stop（含）的元素，这些元素按 step 递增。 返回的元素的类型与参数表达式的类型相同。

支持的类型为：字节、短整型、整型、长整型、日期、时间戳。

start 表达式和 stop 表达式必须解析为相同的类型。 如果 start 表达式和 stop 表达式解析为“日期”或“时间戳”类型，则 step 表达式必须解析为“间隔”类型；否则，step 表达式必须解析为与 start 表达式和 stop 表达式相同的类型。

**参数：**

* start - 表达式。 范围的开始。
* stop - 表达式。 范围的结束（含）。
* step - 可选表达式。 范围的步长。 默认情况下，如果 start 小于或等于 stop，则 step 为 1，否则为 -1。 对于时态序列，step 分别为 1 天和 -1 天。 如果 start 大于 stop，则 step 必须为负数，反之亦然。

**示例：**

```sql
> SELECT sequence(1, 5);
 [1,2,3,4,5]
> SELECT sequence(5, 1);
 [5,4,3,2,1]
> SELECT sequence(to_date('2018-01-01'), to_date('2018-03-01'), interval 1 month);
 [2018-01-01,2018-02-01,2018-03-01]
```

**最低生效版本：** 2.4.0

## <a name="sha"></a>sha

sha(expr) - 返回一个 sha1 哈希值作为 `expr` 的十六进制字符串。

**示例：**

```sql
> SELECT sha('Spark');
 85f5955f4b27a9a4c2aab6ffe5d7189fc298b92c
```

## <a name="sha1"></a>sha1

sha1(expr) - 返回一个 sha1 哈希值作为 `expr` 的十六进制字符串。

**示例：**

```sql
> SELECT sha1('Spark');
 85f5955f4b27a9a4c2aab6ffe5d7189fc298b92c
```

## <a name="sha2"></a>sha2

sha2(expr, bitLength) - 返回 SHA-2 系列的校验和，作为 `expr` 的十六进制字符串。 支持 SHA-224、SHA-256、SHA-384 和 SHA-512。 位长度 0 等效于 256。

**示例：**

```sql
> SELECT sha2('Spark', 256);
 529bc3b07127ecb7e53a4dcf1991d9152c24537d919178022b2c42657f79a26b
```

## <a name="shiftleft"></a>shiftleft

shiftleft(base, expr) - 按位左移。

**示例：**

```sql
> SELECT shiftleft(2, 1);
 4
```

## <a name="shiftright"></a>shiftright

shiftright(base, expr) - 按位（有符号）右移。

**示例：**

```sql
> SELECT shiftright(4, 1);
 2
```

## <a name="shiftrightunsigned"></a>shiftrightunsigned

shiftrightunsigned(base, expr) - 按位（无符号）右移。

**示例：**

```sql
> SELECT shiftrightunsigned(4, 1);
 2
```

## <a name="shuffle"></a>随机选择

shuffle(array) - 返回给定数组的随机排列。

**示例：**

```sql
> SELECT shuffle(array(1, 20, 3, 5));
 [3,1,5,20]
> SELECT shuffle(array(1, 20, null, 3));
 [20,null,3,1]
```

函数为非确定性函数。 **最低生效版本：** 2.4.0

## <a name="sign"></a>签名

sign(expr) - 当 `expr` 为负数、0 或正数时返回 -1.0、0.0 或 1.0。

**示例：**

```sql
> SELECT sign(40);
 1.0
```

## <a name="signum"></a>signum

signum(expr) - 当 `expr` 为负数、0 或正数时返回 -1.0、0.0 或 1.0。

**示例：**

```sql
> SELECT signum(40);
 1.0
```

## <a name="sin"></a>sin

sin(expr) - 返回 `expr` 的正弦，就像通过 `java.lang.Math.sin` 计算得出的一样。

**参数：**

* expr - 以弧度表示的角度

**示例：**

```sql
> SELECT sin(0);
 0.0
```

## <a name="sinh"></a>sinh

sinh(expr) - 返回 `expr` 的双曲正弦，就像通过 `java.lang.Math.sinh` 计算得出的一样。

**参数：**

* expr - 双曲角度

**示例：**

```sql
> SELECT sinh(0);
 0.0
```

## <a name="size"></a>大小

size(expr) - 返回数组或映射的大小。 如果函数的输入为 null，并且 spark.sql.legacy.sizeOfNull 设置为 true，则该函数返回 -1。
如果 spark.sql.legacy.sizeOfNull 设置为 false，则函数会针对 null 输入返回 null。 默认情况下，spark.sql.legacy.sizeOfNull 参数设置为 true。

**示例：**

```sql
> SELECT size(array('b', 'd', 'c', 'a'));
 4
> SELECT size(map('a', 1, 'b', 2));
 2
> SELECT size(NULL);
 -1
```

## <a name="skewness"></a>skewness

skewness(expr) - 返回从组的值计算出的偏度值。

## <a name="slice"></a>切片

slice(x, start, length) - 返回数组 x 的子集，该子集从索引 start 开始（如果 start 为负数，则从索引 end 开始），其长度为指定的长度。

**示例：**

```sql
> SELECT slice(array(1, 2, 3, 4), 2, 2);
 [2,3]
> SELECT slice(array(1, 2, 3, 4), -2, 2);
 [3,4]
```

**最低生效版本：** 2.4.0

## <a name="smallint"></a>smallint

smallint(expr) - 将值 `expr` 强制转换为目标数据类型 `smallint`。

## <a name="sort_array"></a>sort_array

sort_array(array[, ascendingOrder]) - 根据数组元素的自然顺序，按升序或降序对输入数组排序。 Null 元素将放置在按升序返回的数组的开头，或按降序返回的数组的末尾。

**示例：**

```sql
> SELECT sort_array(array('b', 'd', null, 'c', 'a'), true);
 [null,"a","b","c","d"]
```

## <a name="soundex"></a>soundex

soundex(str) - 返回字符串的 Soundex 代码。

**示例：**

```sql
> SELECT soundex('Miller');
 M460
```

## <a name="space"></a>space

space(n) - 返回由 `n` 个空格组成的字符串。

**示例：**

```sql
> SELECT concat(space(2), '1');
   1
```

## <a name="spark_partition_id"></a>spark_partition_id

spark_partition_id() - 返回当前分区 ID。

## <a name="split"></a>split

split(str, regex) - 围绕与 `regex` 匹配的匹配项拆分 `str`。

**示例：**

```sql
> SELECT split('oneAtwoBthreeC', '[ABC]');
 ["one","two","three",""]
```

## <a name="sqrt"></a>sqrt

sqrt(expr) - 返回 `expr` 的平方根。

**示例：**

```sql
> SELECT sqrt(4);
 2.0
```

## <a name="stack"></a>堆栈

stack(n, expr1, …, exprk) - 将 `expr1`、...、`exprk` 分隔成 `n` 行。

**示例：**

```sql
> SELECT stack(2, 1, 2, 3);
 1  2
 3  NULL
```

## <a name="std"></a>std

std(expr) - 返回从组的值计算出的样本标准偏差。

## <a name="stddev"></a>stddev

stddev(expr) - 返回从组的值计算出的样本标准偏差。

## <a name="stddev_pop"></a>stddev_pop

stddev_pop(expr) - 返回从组的值计算出的总体标准偏差。

## <a name="stddev_samp"></a>stddev_samp

stddev_samp(expr) - 返回从组的值计算出的样本标准偏差。

## <a name="str_to_map"></a>str_to_map

str_to_map(text[, pairDelim[, keyValueDelim]]) - 在使用分隔符将文本拆分为键/值对后创建映射。 `pairDelim` 的默认分隔符为“,”，`keyValueDelim` 的为“:”。

**示例：**

```sql
> SELECT str_to_map('a:1,b:2,c:3', ',', ':');
 map("a":"1","b":"2","c":"3")
> SELECT str_to_map('a');
 map("a":null)
```

## <a name="string"></a>string

string(expr) - 将值 `expr` 强制转换为目标数据类型 `string`。

## <a name="struct"></a>struct

struct(col1, col2, col3, …) - 创建具有给定字段值的结构。

## <a name="substr"></a>substr

substr(str, pos[, len]) - 返回 `str` 的从 `pos` 开始且长度为 `len` 的子字符串，或返回从 `pos` 开始且长度为 `len` 的字节数组切片。

**示例：**

```sql
> SELECT substr('Spark SQL', 5);
 k SQL
> SELECT substr('Spark SQL', -3);
 SQL
> SELECT substr('Spark SQL', 5, 1);
 k
```

## <a name="substring"></a>substring

substring(str, pos[, len]) - 返回 `str` 的从 `pos` 开始且长度为 `len` 的子字符串，或返回从 `pos` 开始且长度为 `len` 的字节数组切片。

**示例：**

```sql
> SELECT substring('Spark SQL', 5);
 k SQL
> SELECT substring('Spark SQL', -3);
 SQL
> SELECT substring('Spark SQL', 5, 1);
 k
```

## <a name="substring_index"></a>substring_index

substring_index(str, delim, count) - 返回 `str` 中出现 `count` 次分隔符 `delim` 之前的子字符串。 如果 `count` 为正，则返回最终的分隔符左侧的所有内容（从左侧开始计算）。 如果 `count` 为负，则返回最终的分隔符右侧的所有内容（从右侧开始计算）。 函数 substring_index 在搜索 `delim` 时执行区分大小写的匹配操作。

**示例：**

```sql
> SELECT substring_index('www.apache.org', '.', 2);
 www.apache
```

## <a name="sum"></a>sum

sum(expr) - 返回从组的值计算出的总和值。

## <a name="tan"></a>tan

tan(expr) - 返回 `expr` 的正切，就像通过 `java.lang.Math.tan` 计算得出的一样。

**参数：**

* expr - 以弧度表示的角度

**示例：**

```sql
> SELECT tan(0);
 0.0
```

## <a name="tanh"></a>tanh

tanh(expr) - 返回 `expr` 的双曲正切，就像通过 `java.lang.Math.tanh` 计算得出的一样。

**参数：**

* expr - 双曲角度

**示例：**

```sql
> SELECT tanh(0);
 0.0
```

## <a name="timestamp"></a>timestamp

timestamp(expr) - 将值 `expr` 强制转换为目标数据类型 `timestamp`。

## <a name="tinyint"></a>tinyint

tinyint(expr) - 将值 `expr` 强制转换为目标数据类型 `tinyint`。

## <a name="to_date"></a>to_date

to_date(date_str[, fmt]) - 使用 `fmt` 表达式将 `date_str` 表达式解析为日期。 输入无效时返回 null。 默认情况下，它会根据强制转换规则将表达式转换为日期（如果省略 `fmt`）。

**示例：**

```sql
> SELECT to_date('2009-07-30 04:17:52');
 2009-07-30
> SELECT to_date('2016-12-31', 'yyyy-MM-dd');
 2016-12-31
```

**最低生效版本：** 1.5.0

## <a name="to_json"></a>to_json

to_json(expr[, options]) - 返回具有给定结构值的 JSON 字符串

**示例：**

```sql
> SELECT to_json(named_struct('a', 1, 'b', 2));
```

```
 {"a":1,"b":2}
```sql
> SELECT to_json(named_struct('time', to_timestamp('2015-08-26', 'yyyy-MM-dd')), map('timestampFormat', 'dd/MM/yyyy'));
```

```
 {"time":"26/08/2015"}
```sql
> SELECT to_json(array(named_struct('a', 1, 'b', 2)));
```

```
 [{"a":1,"b":2}]
```sql
> SELECT to_json(map('a', named_struct('b', 1)));
```

```
 {"a":{"b":1}}
```sql
> SELECT to_json(map(named_struct('a', 1),named_struct('b', 2)));
```

```
 {"[1]":{"b":2}}
```sql
> SELECT to_json(map('a', 1));
```

```
 {"a":1}
```sql
> SELECT to_json(array((map('a', 1))));
```

```
 [{"a":1}]
```

**最低生效版本：** 2.2.0

## <a name="to_timestamp"></a>to_timestamp

to_timestamp(timestamp[, fmt]) - 使用 `fmt` 表达式将 `timestamp` 表达式解析为时间戳。 输入无效时返回 null。 默认情况下，它会根据强制转换规则将表达式转换为时间戳（如果省略 `fmt`）。

**示例：**

```sql
> SELECT to_timestamp('2016-12-31 00:12:00');
 2016-12-31 00:12:00
> SELECT to_timestamp('2016-12-31', 'yyyy-MM-dd');
 2016-12-31 00:00:00
```

**最低生效版本：** 2.2.0

## <a name="to_unix_timestamp"></a>to_unix_timestamp

to_unix_timestamp(expr[, pattern]) - 返回给定时间的 UNIX 时间戳。

**示例：**

```sql
> SELECT to_unix_timestamp('2016-04-08', 'yyyy-MM-dd');
 1460041200
```

**最低生效版本：** 1.6.0

## <a name="to_utc_timestamp"></a>to_utc_timestamp

to_utc_timestamp(timestamp, timezone) - 在给定时间戳（例如“2017-07-14 02:40:00.0”）的情况下，将其解释为给定时区的时间，并将该时间呈现为以 UTC 表示的时间戳。 例如，“GMT+1”会生成“2017-07-14 01:40:00.0”。

**示例：**

```sql
> SELECT to_utc_timestamp('2016-08-31', 'Asia/Seoul');
 2016-08-30 15:00:00
```

**最低生效版本：** 1.5.0

## <a name="transform"></a>转换

transform(expr, func) - 使用函数转换数组中的元素。

**示例：**

```sql
> SELECT transform(array(1, 2, 3), x -> x + 1);
 [2,3,4]
> SELECT transform(array(1, 2, 3), (x, i) -> x + i);
 [1,3,5]
```

**最低生效版本：** 2.4.0

## <a name="translate"></a>translate

translate(input, from, to) - 通过将 `from` 字符串中存在的字符替换为 `to` 字符串中的相应字符来转换 `input` 字符串。

**示例：**

```sql
> SELECT translate('AaBbCc', 'abc', '123');
 A1B2C3
```

## <a name="trim"></a>trim

trim(str) - 从 `str` 中删除前导和尾随空格字符。

trim(BOTH trimStr FROM str) - 从 `str` 中删除前导和尾随 `trimStr` 字符

trim(LEADING trimStr FROM str) - 从 `str` 中删除前导 `trimStr` 字符

trim(TRAILING trimStr FROM str) - 从 `str` 中删除尾随 `trimStr` 字符

**参数：**

* str - 字符串表达式
* trimStr - 要剪裁的剪裁字符串字符，默认值为单个空格
* BOTH、FROM - 这些关键字用于指定从字符串两端开始剪裁字符串字符
* LEADING、FROM - 这些关键字用于指定从字符串左端开始剪裁字符串字符
* TRAILING、FROM - 这些关键字用于指定从字符串右端开始剪裁字符串字符

**示例：**

```sql
> SELECT trim('    SparkSQL   ');
 SparkSQL
> SELECT trim('SL', 'SSparkSQLS');
 parkSQ
> SELECT trim(BOTH 'SL' FROM 'SSparkSQLS');
 parkSQ
> SELECT trim(LEADING 'SL' FROM 'SSparkSQLS');
 parkSQLS
> SELECT trim(TRAILING 'SL' FROM 'SSparkSQLS');
 SSparkSQ
```

## <a name="trunc"></a>trunc

trunc(date, fmt) - 返回 `date`，其日期的时间部分已截断到格式模型 `fmt` 所指定的单位。 `fmt` 应该是 [“year”, “yyyy”, “yy”, “mon”, “month”, “mm”] 中的一个项

**示例：**

```sql
> SELECT trunc('2009-02-12', 'MM');
 2009-02-01
> SELECT trunc('2015-10-27', 'YEAR');
 2015-01-01
```

**最低生效版本：** 1.5.0

## <a name="ucase"></a>ucase

ucase(str) - 返回 `str`，其中的所有字符均更改为大写。

**示例：**

```sql
> SELECT ucase('SparkSql');
 SPARKSQL
```

## <a name="unbase64"></a>unbase64

unbase64(str) - 将参数从base64 字符串 `str` 转换为二进制。

**示例：**

```sql
> SELECT unbase64('U3BhcmsgU1FM');
 Spark SQL
```

## <a name="unhex"></a>unhex

unhex(expr) - 将十六进制 `expr` 转换为二进制。

**示例：**

```sql
> SELECT decode(unhex('537061726B2053514C'), 'UTF-8');
 Spark SQL
```

## <a name="unix_timestamp"></a>unix_timestamp

unix_timestamp([expr[, pattern]]) - 返回当前时间或指定时间的 UNIX 时间戳。

**示例：**

```sql
> SELECT unix_timestamp();
 1476884637
> SELECT unix_timestamp('2016-04-08', 'yyyy-MM-dd');
 1460041200
```

**最低生效版本：** 1.5.0

## <a name="upper"></a>upper

upper(str) - 返回 `str`，其中的所有字符均更改为大写。

**示例：**

```sql
> SELECT upper('SparkSql');
 SPARKSQL
```

## <a name="uuid"></a>uuid

uuid() - 返回全局唯一标识符 (UUID) 字符串。 该值以规范 UUID 36 字符字符串形式返回。

**示例：**

```sql
  > SELECT uuid();
   46707d92-02f4-4817-8116-a4c3b23e6266
function is non-deterministic.
```

## <a name="var_pop"></a>var_pop

var_pop(expr) - 返回从组的值计算出的总体方差。

## <a name="var_samp"></a>var_samp

var_samp(expr) - 返回从组的值计算出的样本方差。

## <a name="variance"></a>variance

variance(expr) - 返回从组的值计算出的样本方差。

## <a name="weekday"></a>weekday

weekday(date) - 返回日期/时间戳的星期日期（0 = 星期一，1 = 星期二，...，6 = 星期日）。

**示例：**

```sql
> SELECT weekday('2009-07-30');
 3
```

**最低生效版本：** 2.4.0

## <a name="weekofyear"></a>weekofyear

weekofyear(date) - 返回给定日期是一年中的第几周。 一周被视为从星期一开始，第 1 周的持续时间 > 3 天。

**示例：**

```sql
> SELECT weekofyear('2008-02-20');
 8
```

**最低生效版本：** 1.5.0

## <a name="when"></a>when

CASE WHEN expr1 THEN expr2 [WHEN expr3 THEN expr4]* [ELSE expr5] END - 当 `expr1` = true 时，返回 `expr2`；否则，当 `expr3` = true 时，返回 `expr4`；否则返回 `expr5`。

**参数：**

* expr1、expr3 - 分支条件表达式应该都是布尔类型。
* expr2、expr4、expr5 - 分支值表达式和 else 值表达式应该都是同一类型或能够强制成为某个通用类型。

**示例：**

```sql
> SELECT CASE WHEN 1 > 0 THEN 1 WHEN 2 > 0 THEN 2.0 ELSE 1.2 END;
 1
> SELECT CASE WHEN 1 < 0 THEN 1 WHEN 2 > 0 THEN 2.0 ELSE 1.2 END;
 2
> SELECT CASE WHEN 1 < 0 THEN 1 WHEN 2 < 0 THEN 2.0 END;
 NULL
```

## <a name="window"></a>window

## <a name="xpath"></a>xpath

xpath(xml, xpath) - 返回与 XPath 表达式匹配的 xml 节点内的值的字符串数组。

**示例：**

```sql
> SELECT xpath('<a><b>b1</b><b>b2</b><b>b3</b><c>c1</c><c>c2</c></a>','a/b/text()');
 ['b1','b2','b3']
```

## <a name="xpath_boolean"></a>xpath_boolean

xpath_boolean(xml, xpath) - 如果 XPath 表达式的计算结果为 true，或者找到了匹配的节点，则返回 true。

**示例：**

```sql
> SELECT xpath_boolean('<a><b>1</b></a>','a/b');
 true
```

## <a name="xpath_double"></a>xpath_double

xpath_double(xml, xpath) - 返回双精度值。如果找不到匹配项，则返回值零；如果找到匹配项，但值为非数字值，则返回 NAN。

**示例：**

```sql
> SELECT xpath_double('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3.0
```

## <a name="xpath_float"></a>xpath_float

xpath_float(xml, xpath) - 返回浮点值。如果找不到匹配项，则返回值零；如果找到匹配项，但值为非数字值，则返回 NAN。

**示例：**

```sql
> SELECT xpath_float('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3.0
```

## <a name="xpath_int"></a>xpath_int

xpath_int(xml, xpath) - 返回整数值。如果找不到匹配项，或者找到匹配项，但值为非数字值，则返回值零。

**示例：**

```sql
> SELECT xpath_int('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3
```

## <a name="xpath_long"></a>xpath_long

xpath_long(xml, xpath) - 返回长整型值。如果找不到匹配项，或者找到匹配项，但值为非数字值，则返回值零。

**示例：**

```sql
> SELECT xpath_long('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3
```

## <a name="xpath_number"></a>xpath_number

xpath_number(xml, xpath) - 返回双精度值。如果找不到匹配项，则返回值零；如果找到匹配项，但值为非数字值，则返回 NAN。

**示例：**

```sql
> SELECT xpath_number('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3.0
```

## <a name="xpath_short"></a>xpath_short

xpath_short(xml, xpath) - 返回短整型值。如果找不到匹配项，或者找到匹配项，但值为非数字值，则返回值零。

**示例：**

```sql
> SELECT xpath_short('<a><b>1</b><b>2</b></a>', 'sum(a/b)');
 3
```

## <a name="xpath_string"></a>xpath_string

xpath_string(xml, xpath) - 返回与 XPath 表达式匹配的第一个 xml 节点的文本内容。

**示例：**

```sql
> SELECT xpath_string('<a><b>b</b><c>cc</c></a>','a/c');
 cc
```

## <a name="year"></a>year

year(date) - 返回日期/时间戳的年份部分。

**示例：**

```sql
> SELECT year('2016-07-30');
 2016
```

**最低生效版本：** 1.5.0

## <a name="zip_with"></a>zip_with

zip_with(left, right, func) - 使用函数按元素将两个给定数组合并为单个数组。 如果一个数组较短，则会在应用函数之前，在该数组末尾追加 null，使之与较长数组的长度匹配。

**示例：**

```sql
> SELECT zip_with(array(1, 2, 3), array('a', 'b', 'c'), (x, y) -> (y, x));
```

```
 [{"y":"a","x":1},{"y":"b","x":2},{"y":"c","x":3}]
```

```sql
> SELECT zip_with(array(1, 2), array(3, 4), (x, y) -> x + y);
```

```
 [4,6]
```

```sql
> SELECT zip_with(array('a', 'b', 'c'), array('d', 'e', 'f'), (x, y) -> concat(x, y));
```

```
 ["ad","be","cf"]
```

**最低生效版本：** 2.4.0

## <a name=""></a><a id="f15"> </a>|

expr1 | expr2 - 返回对 `expr1` 和 `expr2` 进行“按位或”运算后的结果。

**示例：**

```sql
> SELECT 3 | 5;
 7
```

## <a name=""></a><a id="f16"> </a>~

~ expr - 返回对 `expr` 进行“按位非”运算后的结果。

**示例：**

```sql
> SELECT ~ 0;
 -1
```