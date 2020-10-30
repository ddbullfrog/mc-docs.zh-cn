---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/20/2020
title: 日期和时间戳 - Azure Databricks
description: 了解如何在 Databricks Runtime 7.0 及更高版本中使用日期和时间戳。
ms.openlocfilehash: 219fa4a8bb1041486251ec0ef99557d2cdc1dbc3
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472783"
---
# <a name="dates-and-timestamps"></a>日期和时间戳

`Date` 和 `Timestamp` 数据类型在 Databricks Runtime 7.0 中进行了重大更改。 本文介绍：

* `Date` 类型和关联的日历。
* `Timestamp` 类型及其与时区关联的方式。 本文还详细说明了时区偏移解决方法，以及 Databricks Runtime 7.0 使用的 Java 8 中新时间 API 的细微行为更改。
* 用于构造日期和时间戳值的 API。
* 在 Apache Spark 驱动程序上收集日期和时间戳对象时的常见错误和最佳做法。

## <a name="dates-and-calendars"></a>日期和日历

`Date` 是年、月、日字段的组合，例如（年=2012，月=12，日=31）。 但是，年、月、日字段的值具有约束，以确保日期值是现实世界中的有效日期。 例如，月的值必须介于 1 到 12 之间，日的值必须介于 1 到 28、29、30 或 31 之间（具体取决于年和月），等等。 `Date` 类型不考虑时区。

### <a name="calendars"></a>日历

`Date` 字段的约束由许多可能的日历之一定义。 有些日历（例如[农历](https://en.wikipedia.org/wiki/Lunar_calendar)）仅在特定区域使用。 有些日历（例如[罗马儒略历](https://en.wikipedia.org/wiki/Julian_calendar)）仅在历史上使用。 实际上存在的国际标准是[公历](https://en.wikipedia.org/wiki/Gregorian_calendar)，几乎在全球各处用于民间用途。 它在 1582 年开始使用，并已扩展为支持 1582 年之前的日期。 此扩展的日历称为[前公历](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar)。

Databricks Runtime 7.0 使用前公历，该日历已被其他数据系统（例如 pandas、R、Apache Arrow）使用。 Databricks Runtime 6.x 及更低版本使用儒略历和公历的组合：对 1582 年之前的日期使用儒略历，对 1582 年之后的日期使用公历。 这是继承自旧版 `java.sql.Date` API，该 API 在 Java 8 中被 `java.time.LocalDate` 替代，后者使用前公历。

## <a name="timestamps-and-time-zones"></a>时间戳和时区

`Timestamp` 类型使用以下新字段对 `Date` 类型进行了扩展：小时、分钟、秒（可以有小数部分）和全球（会话范围内）时区。 它定义具体的时刻。 例如，（年=2012，月=12，日=31，小时=23，分钟=59，秒=59.123456），会话时区为 UTC+01:00。 将时间戳值写出到非文本数据源（如 Parquet）时，这些值只是不带时区信息的时刻（例如 UTC 格式的时间戳）。 如果使用不同的会话时区写入和读取时间戳值，则可能会看到不同值的小时、分钟和秒字段，但它们是同一个具体的时刻。

小时、分钟和秒字段具有标准范围：小时的范围为 0–23，分钟和秒的范围为 0–59。 Spark 支持小数秒，精度达到微秒级。 小数秒的有效范围为 0 到 999,999 微秒。

在任何具体的时刻，你都可以观察到墙上的挂钟显示不同的时间值，具体取决于时区：

> [!div class="mx-imgBorder"]
> ![挂钟](../../../_static/images/spark/wall-clocks.png)

相反，一个挂钟值也可以表示许多不同的时刻。

可以使用时区偏移量明确地将一个本地时间戳绑定到某个时刻。 通常，时区偏移量定义为与格林威治标准时间 (GMT) 或 [UTC+0](https://en.wikipedia.org/wiki/UTC±00:00)（[协调世界时](https://en.wikipedia.org/wiki/UTC%C2%B100:00)）之间相差的偏移量（以小时为单位）。 这种时区信息表示形式消除了歧义，但不方便。 大多数人更愿意指出一个位置，例如 `America/Los_Angeles` 或 `Europe/Paris`。 在时区偏移量的基础上进行这样进一步的抽象可以给生活带来方便，但也增加了复杂性。 例如，你现在必须维护一个特殊的时区数据库，以用于将时区名称映射到偏移量。 由于 Spark 在 JVM 上运行，因此它会将映射委托给 Java 标准库，该库从[互联网号码分配局时区数据库](https://www.iana.org/time-zones) (IANA TZDB) 加载数据。 此外，Java 标准库中的映射机制有一些细微差别，这些差别会影响 Spark 的行为。

从 Java 8 开始的 JDK 公开了一个用于日期时间操作和时区偏移量解决方法的不同 API，Databricks Runtime 7.0 使用该 API。 尽管从时区名称到偏移量的映射具有相同的源 (IANA TZDB)，但它在 Java 8 及更高版本中的实现方式不同于 Java 7。

例如，让我们看看 `America/Los_Angeles` 时区 1883 年之前的一个时间戳：`1883-11-10 00:00:00`。 这一年不同于其他年份，因为在 1883 年 11 月 18 日这一天，所有北美铁路公司都切换到了新的标准时间系统。 使用 Java 7 时间 API，你可以获得本地时间戳的时区偏移量，其形式为 `-08:00`：

```scala
java.time.ZoneId.systemDefault
```

```
res0:java.time.ZoneId = America/Los_Angeles
```

```scala
java.sql.Timestamp.valueOf("1883-11-10 00:00:00").getTimezoneOffset / 60.0
```

```
res1: Double = 8.0
```

等效的 Java 8 API 返回一个不同的结果：

```scala
java.time.ZoneId.of("America/Los_Angeles").getRules.getOffset(java.time.LocalDateTime.parse("1883-11-10T00:00:00"))
```

```
res2: java.time.ZoneOffset = -07:52:58
```

1883 年 11 月 18 日之前，北美的日常时间是一个本地问题。大多数城市和城镇使用某种形式的本地阳历时间，该时间根据已知的时钟（例如教堂尖顶上的时钟或钟表商橱窗中的时钟）进行维护。 这就是你看到如此奇怪的时区偏移量的原因。

此示例表明，Java 8 函数更精确，并且考虑到了来自 IANA TZDB 的历史数据。 切换到 Java 8 时间 API 后，Databricks Runtime 7.0 自然而然地从改进中受益，在解析时区偏移量时更为精确。

对于 `Timestamp` 类型，Databricks Runtime 7.0 还切换到了前公历。 [ISO SQL:2016](https://blog.ansi.org/2018/10/sql-standard-iso-iec-9075-2016-ansi-x3-135/) 标准声明，时间戳的有效范围是从 `0001-01-01 00:00:00` 到 `9999-12-31 23:59:59.999999`。 Databricks Runtime 7.0 完全符合该标准，并且支持此范围内的所有时间戳。 请注意以下子范围（与 Databricks Runtime 6.x 及更低版本相比）：

* `0001-01-01 00:00:00..1582-10-03 23:59:59.999999`. Databricks Runtime 6.x 及更低版本使用儒略历，不符合标准。 Databricks Runtime 7.0 修复了此问题，在对时间戳进行的内部操作（如获取年、月、日等）中应用前公历。由于日历不同，Databricks Runtime 6.x 及更低版本中存在的某些日期不存在于 Databricks Runtime 7.0 中。 例如，1000-02-29 不是有效的日期，因为 1000 年在公历中不是闰年。 另外，对于此时间戳范围，Databricks Runtime 6.x 及更低版本无法正确地将时区名称解析为时区偏移量。
* `1582-10-04 00:00:00..1582-10-14 23:59:59.999999`. 这在 Databricks Runtime 7.0 中是本地时间戳的有效范围，而相比之下，在 Databricks Runtime 6.x 及更低版本中，此类时间戳不存在。
* `1582-10-15 00:00:00..1899-12-31 23:59:59.999999`. Databricks Runtime 7.0 使用 IANA TZDB 中的历史数据正确解析了时区偏移量。 与 Databricks Runtime 7.0 相比，在某些情况下，Databricks Runtime 6.x 及更低版本可能无法正确地从时区名称解析时区偏移量，如前面的示例所示。
* `1900-01-01 00:00:00..2036-12-31 23:59:59.999999`. Databricks Runtime 7.0 和 Databricks Runtime 6.x 均符合 ANSI SQL 标准，并且在日期时间操作（如获取月份日期）中使用公历。
* `2037-01-01 00:00:00..9999-12-31 23:59:59.999999`. Databricks Runtime 6.x 及更低版本可能无法正确解析时区偏移量和夏令时偏移量。 Databricks Runtime 7.0 不存在这个问题。

将时区名称映射到偏移量时，需要考虑的另一个方面是因夏令时 (DST) 操作或切换到其他标准时区偏移量的操作而导致的本地时间戳重叠。 例如，在 2019 年 11 月 3 日 02:00:00，美国的大多数州将时钟向后拨 1 小时，即拨到 01:00:00。 本地时间戳 `2019-11-03 01:30:00 America/Los_Angeles` 可以映射到 `2019-11-03 01:30:00 UTC-08:00` 或 `2019-11-03 01:30:00 UTC-07:00`。 如果你未指定偏移量，而只是设置时区名称（例如 `2019-11-03 01:30:00 America/Los_Angeles`），则 Databricks Runtime 7.0 会采用较早的偏移量（通常对应于“夏季”）。 此行为不同于采用“冬季”偏移量的 Databricks Runtime 6.x 及更低版本。 如果遇到需要将时钟向前拨的情况，则没有有效的偏移量。 进行典型的一小时夏令时更改时，Spark 会将此类时间戳移到对应于“夏季”时间的下一个有效时间戳。

如前面的示例所示，时区名称到偏移量的映射是不明确的，并且不是一对一的。 在可能的情况下，当你构造时间戳时，建议你指定精确的时区偏移量，例如 `2019-11-03 01:30:00 UTC-07:00`。

### <a name="ansi-sql-and-spark-sql-timestamps"></a>ANSI SQL 和 Spark SQL 时间戳

ANSI SQL 标准定义了两种类型的时间戳：

* `TIMESTAMP WITHOUT TIME ZONE` 或 `TIMESTAMP`：（`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`、`SECOND`）形式的本地时间戳。 这些时间戳不绑定到任何时区，是挂钟时间戳。
* `TIMESTAMP WITH TIME ZONE`：（`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`、`SECOND`、`TIMEZONE_HOUR`、`TIMEZONE_MINUTE`）形式的划区时间戳。 这些时间戳表示 UTC 时区中的一个时刻 + 与每个值相关联的时区偏移量（以小时和分钟为单位）。

`TIMESTAMP WITH TIME ZONE` 的时区偏移量不会影响时间戳表示的物理时间点，因为这完全由其他时间戳组件提供的 UTC 时刻表示。 而时区偏移量仅影响显示的时间戳值的默认行为、日期/时间组件提取（例如 `EXTRACT`）以及需要了解时区的其他操作（例如，将月份添加到时间戳）。

Spark SQL 将时间戳类型定义为 `TIMESTAMP WITH SESSION TIME ZONE`，这是多个字段（`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`、`SECOND`、`SESSION TZ`）的组合，其中的 `YEAR` 到 `SECOND` 字段用于标识 UTC 时区中的时刻，而其中的 SESSION TZ 则取自 SQL 配置 spark.sql.session.timeZone。 会话时区可以设置为：

* 时区偏移量 `(+|-)HH:mm`。 可以通过这种形式明确地定义物理时间点。
* 采用区域 ID `area/city` 形式的时区名称，例如 `America/Los_Angeles`。 这种形式的时区信息受前面所述的一些问题（如本地时间戳重叠）的影响。 但是，每个 UTC 时刻都明确地与任意区域 ID 的一个时区偏移量相关联。因此，每个带有基于区域 ID 的时区的时间戳都可以明确地转换为具有时区偏移量的时间戳。 默认情况下，会话时区设置为 Java 虚拟机的默认时区。

Spark `TIMESTAMP WITH SESSION TIME ZONE` 不同于：

* `TIMESTAMP WITHOUT TIME ZONE`，因为此类型的值可以映射到多个物理时刻，但 `TIMESTAMP WITH SESSION TIME ZONE` 的任何值都是具体的物理时刻。 可以通过在所有会话中使用一个固定的时区偏移量（例如 UTC+0）来模拟 SQL 类型。 在这种情况下，可以将 UTC 时间戳视为本地时间戳。
* `TIMESTAMP WITH TIME ZONE`，因为根据 SQL 标准，该类型的列值可以有不同的时区偏移量。 这不受 Spark SQL 支持。

你应该注意到，与全球（会话范围内）时区相关联的时间戳不是 Spark SQL 的创新。 RDBMS（例如 Oracle）为时间戳提供类似类型：`TIMESTAMP WITH LOCAL TIME ZONE`。

## <a name="construct-dates-and-timestamps"></a>构造时间和时间戳

Spark SQL 为构造日期和时间戳值提供了几种方法：

* 不带参数的默认构造函数：`CURRENT_TIMESTAMP()` 和 `CURRENT_DATE()`。
* 基于其他基元 Spark SQL 类型，如 `INT`、`LONG` 和 `STRING`
* 基于 Python 日期/时间或 Java 类 `java.time.LocalDate`/`Instant` 等外部类型。
* 从数据源（例如 CSV、JSON、Avro、Parquet、ORC 等）进行的反序列化。

Databricks Runtime 7.0 中引入的函数 `MAKE_DATE` 采用三个参数（`YEAR`、`MONTH` 和 `DAY`），构造了一个 `DATE` 值。 只要可能，所有输入参数都会隐式转换为 `INT` 类型。 此函数会检查生成的日期是否是前公历中的有效日期，在不是的情况下会返回 `NULL`。 例如： 。

```python
spark.createDataFrame([(2020, 6, 26), (1000, 2, 29), (-44, 1, 1)],['Y', 'M', 'D']).createTempView('YMD')
df = sql('select make_date(Y, M, D) as date from YMD')
df.printSchema()
```

```
root
|-- date: date (nullable = true)
```

若要输出数据帧内容，请调用 `show()` 操作。该操作会在执行程序上将日期转换为字符串，并将字符串传输给驱动程序，以便在控制台上输出它们：

```python
df.show()
```

```
+-----------+
|       date|
+-----------+
| 2020-06-26|
|       null|
|-0044-01-01|
+-----------+
```

同样，你可以使用 `MAKE_TIMESTAMP` 函数构造时间戳值。 与 `MAKE_DATE` 一样，它对日期字段执行相同的验证，另外还接受时间字段 HOUR (0-23)、MINUTE (0-59) 和 SECOND (0-60)。 SECOND 的类型为 Decimal（精度 = 8，刻度 = 6），因为可以传递小数部分达到微秒精度的秒。 例如： 。

```python
df = spark.createDataFrame([(2020, 6, 28, 10, 31, 30.123456), \
(1582, 10, 10, 0, 1, 2.0001), (2019, 2, 29, 9, 29, 1.0)],['YEAR', 'MONTH', 'DAY', 'HOUR', 'MINUTE', 'SECOND'])
df.show()
```

```
+----+-----+---+----+------+---------+
|YEAR|MONTH|DAY|HOUR|MINUTE|   SECOND|
+----+-----+---+----+------+---------+
|2020|    6| 28|  10|    31|30.123456|
|1582|   10| 10|   0|     1|   2.0001|
|2019|    2| 29|   9|    29|      1.0|
+----+-----+---+----+------+---------+
```

```python
df.selectExpr("make_timestamp(YEAR, MONTH, DAY, HOUR, MINUTE, SECOND) as MAKE_TIMESTAMP")
ts.printSchema()
```

```
root
|-- MAKE_TIMESTAMP: timestamp (nullable = true)
```

对于日期，则使用 show() 操作输出 ts 数据帧的内容。 同样，`show()` 会将时间戳转换为字符串，但现在它会考虑到 SQL 配置 `spark.sql.session.timeZone` 所定义的会话时区。

```python
ts.show(truncate=False)
```

```
+--------------------------+
|MAKE_TIMESTAMP            |
+--------------------------+
|2020-06-28 10:31:30.123456|
|1582-10-10 00:01:02.0001  |
|null                      |
+--------------------------+
```

Spark 无法创建最后一个时间戳，因为此日期无效：2019 不是闰年。

你可能注意到前面的示例中没有时区信息。 在这种情况下，Spark 使用 SQL 配置 `spark.sql.session.timeZone` 中的时区，将其应用于函数调用。 你还可以通过将时区作为 `MAKE_TIMESTAMP` 的最后一个参数进行传递来选取不同的时区。 以下是示例：

```python
df = spark.createDataFrame([(2020, 6, 28, 10, 31, 30, 'UTC'),(1582, 10, 10, 0, 1, 2, 'America/Los_Angeles'), \
(2019, 2, 28, 9, 29, 1, 'Europe/Moscow')], ['YEAR', 'MONTH', 'DAY', 'HOUR', 'MINUTE', 'SECOND', 'TZ'])
df = df.selectExpr('make_timestamp(YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, TZ) as MAKE_TIMESTAMP')
df = df.selectExpr("date_format(MAKE_TIMESTAMP, 'yyyy-MM-dd HH:mm:SS VV') AS TIMESTAMP_STRING")
df.show(truncate=False)
```

```
+---------------------------------+
|TIMESTAMP_STRING                 |
+---------------------------------+
|2020-06-28 13:31:00 Europe/Moscow|
|1582-10-10 10:24:00 Europe/Moscow|
|2019-02-28 09:29:00 Europe/Moscow|
+---------------------------------+
```

如示例所示，Spark 会考虑到指定的时区，但会将所有本地时间戳调整成会话时区。 传递到 `MAKE_TIMESTAMP` 函数的原始时区会丢失，因为 `TIMESTAMP WITH SESSION TIME ZONE` 类型假定所有值都属于一个时区，它不会为每个值都存储一个时区。 根据 `TIMESTAMP WITH SESSION TIME ZONE` 的定义，Spark 会将本地时间戳按 UTC 时区存储，而在提取日期-时间字段或将时间戳转换为字符串时则使用会话时区。

此外，可以使用强制转换基于 LONG 类型构造时间戳。 如果 LONG 列包含自 epoch 1970-01-01 00:00:00Z 以来的秒数，则可将其强制转换为 Spark SQL `TIMESTAMP`：

```sql
select CAST(-123456789 AS TIMESTAMP);
1966-02-02 05:26:51
```

遗憾的是，此方法不允许指定秒的小数部分。

另一种方法是基于 `STRING` 类型的值构造日期和时间戳。 可以使用特殊关键字创建文本：

```sql
select timestamp '2020-06-28 22:17:33.123456 Europe/Amsterdam', date '2020-07-01';
2020-06-28 23:17:33.123456        2020-07-01
```

也可使用可应用于列中所有值的强制转换：

```sql
select cast('2020-06-28 22:17:33.123456 Europe/Amsterdam' as timestamp), cast('2020-07-01' as date);
2020-06-28 23:17:33.123456        2020-07-01
```

如果输入的字符串中省略了时区，则会将输入时间戳字符串解释为指定的时区或会话时区中的本地时间戳。 使用 `to_timestamp()` 函数可以将模式异常的字符串转换为时间戳。 [Datetime Patterns for Formatting and Parsing](http://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)（适用于格式设置和分析的日期/时间模式）中介绍了支持的模式：

```sql
select to_timestamp('28/6/2020 22.17.33', 'dd/M/yyyy HH.mm.ss');
2020-06-28 22:17:33
```

如果未指定模式，则此函数的行为与 `CAST` 类似。

为了可用性，Spark SQL 会识别所有接受字符串并返回时间戳或日期的方法中的特殊字符串值：

* `epoch` 是日期 `1970-01-01` 或时间戳 `1970-01-01 00:00:00Z` 的别名。
* `now` 是会话时区的当前时间戳或日期。 在单个查询中，它始终产生同一结果。
* `today` 是 `TIMESTAMP` 类型的当前日期的起点，或者就是 `DATE` 类型的当前日期。
* `tomorrow` 是时间戳的下一天的起点，或者就是 `DATE` 类型的下一天。
* `yesterday` 是 `TIMESTAMP` 类型的当前日期之前的一天或其起点。

例如： 。

```sql
select timestamp 'yesterday', timestamp 'today', timestamp 'now', timestamp 'tomorrow';
2020-06-27 00:00:00        2020-06-28 00:00:00        2020-06-28 23:07:07.18        2020-06-29 00:00:00
select date 'yesterday', date 'today', date 'now', date 'tomorrow';
2020-06-27        2020-06-28        2020-06-28        2020-06-29
```

Spark 允许从驱动程序端现有的外部对象集合创建 `Datasets`，并创建相应类型的列。 Spark 将外部类型的实例转换为语义上等效的内部表示形式。 例如，若要从 Python 集合创建带有 `DATE` 和 `TIMESTAMP` 列的 `Dataset`，可以使用：

```python
import datetime
df = spark.createDataFrame([(datetime.datetime(2020, 7, 1, 0, 0, 0), datetime.date(2020, 7, 1))], ['timestamp', 'date'])
df.show()
```

```
+-------------------+----------+
|          timestamp|      date|
+-------------------+----------+
|2020-07-01 00:00:00|2020-07-01|
+-------------------+----------+
```

PySpark 使用系统时区在驱动程序端将 Python 的日期-时间对象转换为内部 Spark SQL 表示形式，该表示形式可能不同于 Spark 的会话时区设置 `spark.sql.session.timeZone`。 内部值不包含有关原始时区的信息。 针对并行化日期和时间戳值的未来操作仅会根据 `TIMESTAMP WITH SESSION TIME ZONE` 类型定义考虑 Spark SQL 会话时区。

类似地，Spark 会在 Java 和 Scala API 中将以下类型识别为外部日期-时间类型：

* `java.sql.Date` 和 `java.time.LocalDate`，作为 `DATE` 类型的外部类型
* `java.sql.Timestamp` 和 `java.time.Instant`，适用于 `TIMESTAMP` 类型。

`java.sql.*` 和 `java.time.*` 类型之间存在差异。 Java 8 中添加了 `java.time.LocalDate` 和 `java.time.Instant`，这些类型基于前公历 – Databricks Runtime 7.0 及更高版本所使用的日历。 `java.sql.Date` 和 `java.sql.Timestamp` 下有另一个日历 – 混合日历（自 1582-10-15 以来使用的儒略历 + 公历），这与 Databricks Runtime 6.x 及更低版本使用的旧日历相同。 由于日历系统不同，Spark 必须在转换为内部 Spark SQL 表示形式期间执行其他操作，并将输入日期/时间戳从一个日历变基为另一个日历。 对于 1900 年后的新式时间戳，该变基操作的开销很小，并且它可能对旧时间戳更有意义。

以下示例演示如何从 Scala 集合创建时间戳。 第一个示例根据一个字符串构造 `java.sql.Timestamp` 对象。 `valueOf` 方法将输入字符串解释为默认 JVM 时区中的本地时间戳，该时区可能不同于 Spark 的会话时区。 如果需要在特定时区构造 `java.sql.Timestamp` 或 `java.sql.Date` 的实例，请查看 [java.text.SimpleDateFormat](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html)（及其方法 `setTimeZone`）或 [java.util.Calendar](https://docs.oracle.com/javase/7/docs/api/java/util/Calendar.html)。

```scala
Seq(java.sql.Timestamp.valueOf("2020-06-29 22:41:30"), new java.sql.Timestamp(0)).toDF("ts").show(false)
```

```
+-------------------+
|ts                 |
+-------------------+
|2020-06-29 22:41:30|
|1970-01-01 03:00:00|
+-------------------+
```

```scala
Seq(java.time.Instant.ofEpochSecond(-12219261484L), java.time.Instant.EPOCH).toDF("ts").show
```

```
+-------------------+
|                 ts|
+-------------------+
|1582-10-15 11:12:13|
|1970-01-01 03:00:00|
+-------------------+
```

类似地，你可以基于 `java.sql.Date` 或 `java.sql.LocalDate` 的集合创建 `DATE` 列。 `java.sql.LocalDate` 实例的并行化完全独立于 Spark 的会话或 JVM 默认时区，但 `java.sql.Date` 实例的并行化有所不同。 有一些细微差别：

1. `java.sql.Date` 实例表示驱动程序上默认 JVM 时区的本地日期。
2. 若要正确地转换为 Spark SQL 值，驱动程序和执行程序上的默认 JVM 时区必须相同。

```scala
Seq(java.time.LocalDate.of(2020, 2, 29), java.time.LocalDate.now).toDF("date").show
```

```
+----------+
|      date|
+----------+
|2020-02-29|
|2020-06-29|
+----------+
```

为了避免任何与日历和时区相关的问题，建议在对时间戳或日期的 Java/Scala 集合进行并行化时将 Java 8 类型 `java.sql.LocalDate`/`Instant` 用作外部类型。

## <a name="collect-dates-and-timestamps"></a>收集时间和时间戳

并行化的反向操作是将日期和时间戳从执行程序收集回驱动程序，并返回外部类型的集合。 对于上面的示例，你可以使用 `collect()` 操作将 `DataFrame` 拉取回驱动程序：

```scala
df.collect()
```

```
[Row(timestamp=datetime.datetime(2020, 7, 1, 0, 0), date=datetime.date(2020, 7, 1))]
```

Spark 会将日期和时间戳列的内部值作为 UTC 时区中的时刻从执行程序传输到驱动程序，并在驱动程序的系统时区中执行转换到 Python 日期/时间对象，而不使用 Spark SQL 会话时区。 `collect()` 与上一部分所述的 `show()` 操作不同。 `show()` 在将时间戳转换为字符串时使用会话时区，并收集驱动程序上生成的字符串。

在 Java 和 Scala API 中，Spark 默认执行以下转换：

* 将 Spark SQL `DATE` 值转换为 `java.sql.Date` 的实例。
* 将 Spark SQL `TIMESTAMP` 值转换为 `java.sql.Timestamp` 的实例。

这两种转换都在驱动程序的默认 JVM 时区中执行。 若要通过这种方式获取使用 `Date.getDay()`、`getHour()` 等方法以及使用 Spark SQL 函数 `DAY`、`HOUR` 获取的日期-时间字段，驱动程序上的默认 JVM 时区和执行程序上的会话时区应相同。

类似于通过 `java.sql.Date`/`Timestamp` 创建日期/时间戳，Databricks Runtime 7.0 会执行从前公历变基到混合日历（儒略历 + 公历）。 对于新式日期（1582 年之后的日期）和时间戳（1900 年之后的时间戳），此操作几乎没有任何开销，但对于老式的日期和时间戳，它可能会产生一定的开销。

你可以避免这种与日历相关的问题，并要求 Spark 返回从 Java 8 开始增加的 `java.time` 类型。 如果将 SQL 配置 `spark.sql.datetime.java8API.enabled` 设置为 true，则 `Dataset.collect()` 操作会返回：

* `java.time.LocalDate`（适用于 Spark SQL `DATE` 类型）
* `java.time.Instant`（适用于 Spark SQL `TIMESTAMP` 类型）

现在，转换不会遇到与日历相关的问题了，因为 Java 8 类型和 Databricks Runtime 7.0 及更高版本都基于前公历。 `collect()` 操作不依赖于默认的 JVM 时区。 时间戳转换根本不依赖于时区。 日期转换使用 SQL 配置 `spark.sql.session.timeZone` 中的会话时区。 例如，请考虑一个具有 `DATE` 和 `TIMESTAMP` 列的 `Dataset`，其中的默认 JVM 时区设置为 `Europe/Moscow`，会话时区设置为 `America/Los_Angeles`。

```scala
java.util.TimeZone.getDefault
```

```
res1: java.util.TimeZone = sun.util.calendar.ZoneInfo[id="Europe/Moscow",...]
```

```scala
spark.conf.get("spark.sql.session.timeZone")
```

```
res2: String = America/Los_Angeles
```

```scala
df.show
```

```
+-------------------+----------+
|          timestamp|      date|
+-------------------+----------+
|2020-07-01 00:00:00|2020-07-01|
+-------------------+----------+
```

`show()` 操作在会话时间 `America/Los_Angeles` 输出时间戳，但如果你收集 `Dataset`，则它会转换为 `java.sql.Timestamp`，并且 `toString` 方法会输出 `Europe/Moscow`：

```scala
df.collect()
```

```
res16: Array[org.apache.spark.sql.Row] = Array([2020-07-01 10:00:00.0,2020-07-01])
```

```scala
df.collect()(0).getAs[java.sql.Timestamp](0).toString
```

```
res18: java.sql.Timestamp = 2020-07-01 10:00:00.0
```

实际上，本地时间戳 2020-07-01 00:00:00 是 2020-07-01T07:00:00Z (UTC)。 如果启用 Java 8 API 并收集数据集，则可以观察到：

```scala
df.collect()
```

```
res27: Array[org.apache.spark.sql.Row] = Array([2020-07-01T07:00:00Z,2020-07-01])
```

你可以独立于全球 JVM 时区将 `java.time.Instant` 对象转换为任意本地时间戳。 这是 `java.time.Instant` 相对于 `java.sql.Timestamp` 的优点之一。 前者需要更改全球 JVM 设置，这会影响同一 JVM 上的其他时间戳。 因此，如果应用程序处理不同时区的日期或时间戳，并且应用程序在使用 Java 或 Scala `Dataset.collect()` API 将数据收集到驱动程序时不会相互冲突，建议使用 SQL 配置 `spark.sql.datetime.java8API.enabled` 切换到 Java 8 API。