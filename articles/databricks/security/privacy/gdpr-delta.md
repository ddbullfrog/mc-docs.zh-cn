---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/04/2020
title: 最佳做法 - 使用 Delta Lake 的 GDPR 和 CCPA 合规性 - Azure Databricks
description: 了解如何使用 Delta Lake 来满足数据湖的 GDPR 和 CCPA 要求。
ms.openlocfilehash: 9ea7050f14d39b2f45151fc8c965e9f24839459b
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937740"
---
# <a name="best-practices-gdpr-and-ccpa-compliance-using-delta-lake"></a>最佳做法：使用 Delta Lake 实现 GDPR 和 CCPA 符合性

本文介绍如何使用 Azure Databricks 上的 Delta Lake 管理数据湖的《一般数据保护条例》(GDPR) 和《加州消费者隐私法》(CCPA) 合规性。 由于 Delta Lake 添加了一个事务层，该事务层在数据湖之上提供结构化的数据管理，因此它可以极大地简化查找和删除个人信息（也称为“个人数据”）的功能并加快响应消费者 GDPR 或 CCPA 要求的速度。

## <a name="the-challenge"></a>难题

组织可以在云中管理数百 TB 的个人信息。 将这些数据集引入 GDPR 和 CCPA 合规性至关重要，但这可能是一个巨大的挑战，尤其是对于存储在数据湖中的大型数据集。

通常，面临的挑战如下：

* 如果云中有大量（PB 级）数据，用户数据可能会跨多个数据集和位置进行存储和分发。
* 用于查找特定用户数据的点查询或即席查询开销较高（类似于大海捞针），因为它通常需要进行全表扫描。 对 GDPR/CCPA 的合规性采用蛮力方法可能会导致多个作业在不同的表上运行，从而导致数周的工程和运营工作量。
* 数据湖本质上是只能追加的，本身不支持执行行级别“删除”或“更新”操作的功能，这意味着必须重写数据分区。 典型的数据湖产品/服务不提供 ACID 事务功能或用于查找相关数据的有效方法。 此外，读/写一致性也是一个问题：从数据湖编辑用户数据时，应保护数据读取过程，使其免受传统 RDBMS 方式的实质影响。
* 鉴于数据湖设计上支持可用性和具有最终一致性的分区容错，数据湖中的数据清理具有挑战性。 需要强制执行严格的做法和标准来确保干净的数据。

因此，以这种规模管理用户数据的组织最后通常会编写难以计算、昂贵且耗时的数据管道来应对 GDPR 和 CCPA。 例如，可以将部分数据湖上传到专有数据仓库技术中，在该技术中执行与 GDPR 和 CCPA 合规性相关的删除活动。 通过强制执行多个数据副本，这会增加复杂度并降低数据保真度。 此外，将数据从此类仓库技术导出回数据湖可能需要重新优化，以提高查询性能。 这也会导致创建和维护多个数据副本。

## <a name="how-delta-lake-addresses-the-challenge"></a>Delta Lake 如何应对挑战

为了解决上面列出的问题，使数据湖符合 GDPR 和 CCPA 的最佳方法要求：

* 将个人信息元素 (`identifiers`)“[假名化](https://en.wikipedia.org/wiki/Pseudonymization)”或可逆地将其标记为无法从外部识别的键 (`pseudonyms`)。
* 以链接到假名而非标识符的方式存储信息；
* 保持对标识符和假名组合的严格访问和使用策略；
* 用于在时间线上删除原始数据的管道或 Bucket 策略，可帮助你遵守适用法律；
* 构建管道以查找和删除标识符，从而破坏假名和标识符之间的链接
* ACID 功能覆盖在数据湖的顶部，以防止在对数据湖执行删除或更新操作时对读取器造成负面影响。
* 高性能管道，例如，支持在 10 分钟内清理 5 TB 数据。

Delta Lake 是一种非常有效的工具，可用于满足这些 GDPR 和 CCPA 合规性要求，因为它的结构化数据管理系统为数据湖增加了事务性功能。 Delta Lake 的结构完整、大小合理、索引完好且启用了统计信息的数据集，可使用标准 SQL DML 语句（如 `DELETE`、`UPDATE` 和 `MERGE INTO`）快速轻松地搜索、修改和清除数据。

以下各节中描述的两个用例说明了如何将现有数据转换为 Delta Lake，以及如何快速有效地删除和清理个人信息。 本文还推荐了用于对个人信息进行假名化和使用 Delta Lake 改进查询性能的选项。

## <a name="delete-personal-data"></a>删除个人数据

此用例演示从数据湖删除个人数据时，Delta Lake 的效率如何。

### <a name="the-sample-dataset"></a>示例数据集

本文中描述的工作流引用了数据库 `gdpr`，该数据库包含具有 65,000,000 行和许多不同客户 ID 的示例数据集，数据总计 3.228 GB。 客户个人信息捕获在该数据库的 `customers` 表中。

`gdpr.customers` 表的架构为：

```
|-- c_customer_sk: integer (nullable = true)
|-- c_customer_id: string (nullable = true)
|-- c_current_cdemo_sk: integer (nullable = true)
|-- c_current_hdemo_sk: integer (nullable = true)
|-- c_current_addr_sk: integer (nullable = true)
|-- c_first_shipto_date_sk: integer (nullable = true)
|-- c_first_sales_date_sk: integer (nullable = true)
|-- c_salutation: string (nullable = true)
|-- c_first_name: string (nullable = true)
|-- c_last_name: string (nullable = true)
|-- c_preferred_cust_flag: string (nullable = true)
|-- c_birth_day: integer (nullable = true)
|-- c_birth_month: integer (nullable = true)
|-- c_birth_year: integer (nullable = true)
|-- c_birth_country: string (nullable = true)
|-- c_email_address: string (nullable = true)
|-- c_last_review_date: string (nullable = true)
```

每个 GDPR 和 CCPA 要求忘记的客户列表来自使用在线门户填充的事务数据库表 `gdpr.customer_delete_keys`。 要删除的键（不同的用户）约占从 `gdpr.customers` 中的原始数据集中采样的原始键的 10% (337.615 MB)。

`gdpr.customer_delete_keys` 表的架构包含以下字段：

```
|-- c_customer_sk: integer (nullable = true)
|-- c_customer_id: string (nullable = true)
```

键 `c_customer_id` 标识要删除的客户。

### <a name="step-1-convert-tables-to-delta-format"></a>步骤 1：将表格转换为 Delta 格式

若要开始使用 Delta Lake，需要引入原始数据（Parquet、CSV 和 JSON 等）并将其写为托管 Delta 表。 如果数据已采用 Parquet 格式，则可以使用 [CONVERT TO DELTA](../../spark/latest/spark-sql/language-manual/convert-to-delta.md) 将 Parquet 文件转换为 Delta 表，而无需重写任何数据。 否则，可以使用熟悉的 Apache Spark API 将[格式重写为 Delta](../../delta/quick-start.md#create-a-table)。 由于 Delta Lake 使用的 Parquet 是一种开放文件格式，因此转换后的数据将不会被锁定：可以按需快速轻松地将数据转换回其他格式。

本示例转换 `gdpr` 数据库中的 Parquet 表 `customers`。

```sql
CONVERT TO DELTA gdpr.customers
```

### <a name="step-2-perform-deletes"></a><a id="gdpr-delete"> </a><a id="step-2-perform-deletes"> </a>步骤2：执行删除操作

在将表转换为 Delta Lake 之后，可以删除那些请求被忘记的用户的个人信息。

> [!NOTE]
>
> 以下示例涉及从 `customers` 表中直接删除客户个人数据。 更好的做法是，（在接收数据主体请求之前）对工作表中的所有客户个人信息进行假名化，并从“查找表”中删除将客户映射为假名的客户条目，同时确保工作表中的数据不能用于重新构建客户的身份。 有关详细信息，请参阅[对数据进行假名化](#pseudonymize-data)。

> [!NOTE]
>
> 以下示例引用性能数字，以说明某些性能选项的影响。 这些数字记录在上述数据集中的一个群集中，该群集具有 3 个工作器节点，其中每个工作器节点都具有 90 GB 内存和 12 个核心；驱动程序具有 30 GB 内存和 4 个核心。

下面是一个简单的 Delta Lake [DELETE FROM](../../spark/latest/spark-sql/language-manual/delete-from.md) 操作，从示例 `gdpr.customers` 表中删除 `customer_delete_keys` 表中包含的客户：

```sql
DELETE FROM `gdpr.customers` AS t1 WHERE EXISTS (SELECT c_customer_id FROM gdpr.customer_delete_keys WHERE t1.c_customer_id = c_customer_id)
```

在测试过程中，完成此操作所用的时间太长：查找文件花费了 32 秒，重写文件花费了 2.6 分钟。为了减少查找相关文件的时间，可以增加广播阈值：

```sql
set spark.sql.autoBroadcastJoinThreshold = 104857600;
```

此广播提示与其他表或视图联接时，指示 Spark 广播每个指定的表。 此设置将文件查找时间减少到 8 秒，并将写入时间减少到 1.6 分钟。

可以使用 Delta Lake [Z 排序（多维聚类分析）](../../delta/optimizations/file-mgmt.md#z-ordering-multi-dimensional-clustering)来进一步提高性能。 Z 排序可创建基于范围分区的数据排列，并在 Delta 表中为此信息编制索引。 Delta Lake 使用此 Z-索引查找受 `DELETE` 操作影响的文件。

若要利用 Z 排序，必须了解希望删除的数据在目标表中的分布情况。 例如，如果数据分布在数据集的 90% 的文件中，即使是几个键，也将重写超过 90% 的数据。 按相关键列进行 Z 排序可减少涉及的文件数，并使重写效率更高。

在这种情况下，应在运行删除之前按 `c_customer_id` 列进行 Z 排序：

```sql
OPTIMIZE gdpr.customers Z-ORDER BY c_customer_id
```

进行 Z 排序后，查找文件需要 7 秒钟，而写入则减少到 50 秒。

### <a name="step-3-clean-up-stale-data"></a>步骤 3：清理过时数据

根据使用者请求在多久之后删除数据以及根据基础数据湖，你可能需要删除表历史记录和基础原始数据。

默认情况下，Delta Lake 保留表历史记录 30 天，并使其可用于[“按时间顺序查看”](../../delta/delta-batch.md#query-an-older-snapshot-of-a-table-time-travel)和回退。 这意味着，即使已从 Delta 表中删除了个人信息，组织中的用户也可以查看该历史数据并将其回退到仍存储个人信息的表版本。 如果确定 GDPR 或 CCPA 合规性要求在默认保持期结束之前，这些过时记录不可用于查询，则可以使用 [VACUUM](../../delta/delta-utility.md#vacuum) 函数删除 Delta 表不再引用且早于指定保留阈值的文件。 使用 VACUUM 命令删除表历史记录后，所有用户都将无法查看和回退该历史记录。

若要删除所有要求删除其信息的客户，然后删除 7 天以上的所有表历史记录，只需运行：

```sql
VACUUM gdpr.customers
```

若要删除 7 天以内的项目，请使用 `RETAIN num HOURS` 选项：

```sql
VACUUM gdpr.customers RETAIN 100 HOURS
```

此外，如果使用 Spark API 创建了 Delta 表以[将非 Parquet 文件重写为 Delta](../../delta/quick-start.md#create-a-table)（而不是将 Parquet 文件就地转换为 Delta Lake），则原始数据可能仍包含已删除或已匿名的个人信息。 Databricks 建议你与云提供商建立 30 天或更短时间的保留策略，以自动删除原始数据。

## <a name="pseudonymize-data"></a>对数据进行假名化

虽然上述删除方法可以严格地允许组织遵守 GDPR 和 CCPA 要求来删除个人信息，但它具有许多缺点。  其中一个缺点便是，收到有效的删除请求后，GDPR 不允许对个人信息进行任何其他处理。  因此，如果在收到数据主体请求之前，数据未以假名化的方式进行存储，即用人工标识符或假名来替换个人身份信息，则你只需负责删除所有链接的信息。  但如果先前已对基础数据进行了假名化，则可以通过简单地销毁将标识符链接到假名的任何记录来实现删除责任（假设剩余数据本身不可识别），并且可以保留剩余的数据。

在典型的假名化方案中，需要保留一个安全的“查找表”，该表将客户的个人标识符（姓名、电子邮件地址等）映射到该假名。 这样，不仅可以简化删除，还允许暂时“还原”用户标识，以随着时间的推移更新用户数据，匿名化方案中则不具备这样的优点。在这种情况下，根据定义，客户的标识永远无法还原，所有客户数据都是静态数据和历史数据。

对于简单的假名化示例，请考虑使用在删除示例中更新的客户表。 在假名化方案中，可以创建一个 `gdpr.customers_lookup` 表，其中包含可用于标识客户的所有客户数据，以及用于假名化电子邮件地址的附加列。  现在，可以将伪电子邮件地址用作引用客户的任何数据表中的密钥，当请求忘记此信息时，只需从 `gdpr.customers_lookup` 表中删除该信息即可，其余信息就会永远不可识别。

`gdpr.customers_lookup` 表的架构为：

```
|-- c_customer_id: string (nullable = true)
|-- c_email_address: string (nullable = true)
|-- c_email_address_pseudonym: string (nullable = true)
|-- c_first_name: string (nullable = true)
|-- c_last_name: string (nullable = true)
```

> [!div class="mx-imgBorder"]
> ![客户查找表](../../_static/images/privacy/gdpr-lookup.png)

此示例将无法用于标识客户的其余客户数据放在名为 `gdpr.customers_pseudo` 的假名表中：

```
|-- c_email_address_pseudonym: string (nullable = true)
|-- c_customer_sk: integer (nullable = true)
|-- c_current_cdemo_sk: integer (nullable = true)
|-- c_current_hdemo_sk: integer (nullable = true)
|-- c_current_addr_sk: integer (nullable = true)
|-- c_first_shipto_date_sk: integer (nullable = true)
|-- c_first_sales_date_sk: integer (nullable = true)
|-- c_salutation: string (nullable = true)
|-- c_preferred_cust_flag: string (nullable = true)
|-- c_birth_year: integer (nullable = true)
|-- c_birth_country: string (nullable = true)
|-- c_last_review_date: string (nullable = true)
```

> [!div class="mx-imgBorder"]
> ![客户假名表](../../_static/images/privacy/gdpr-pseudo.png)

### <a name="use-delta-lake-to-pseudonymize-customer-data"></a>使用 Delta Lake 对客户数据进行假名化

假名化个人信息的一种有效方法是使用记住的一个或多个加盐进行单向加密哈希处理和加盐处理。 哈希将数据转换为固定长度的指纹，该指纹无法通过计算反转。 加盐会在数据中添加随机字符串，该字符串将进行哈希处理，用于阻止使用包含数百万个已知电子邮件地址或密码哈希值的查找表或“彩虹”表的攻击者。

在进行哈希处理之前，可以通过添加随机机密字符串字面量来对列 `c_email_address` 进行加盐处理。 可以使用 Azure Databricks 机密存储此机密字符串，为盐增加额外的安全性。 如果未经授权的 Azure Databricks 用户尝试访问机密，他们将看到已编辑的值。

```scala
dbutils.secrets.get(scope = "salt", key = "useremail")
```

```console
res0: String = [REDACTED]
```

> [!NOTE]
>
> 下面是说明进行加盐处理的简单示例。 为所有客户密钥使用相同的盐并不能有效地缓解攻击，这只会使客户密钥更长。 一种更为安全的方法是为每个用户生成随机盐。  请参阅[使假名化更可靠](#pseudonymization-stronger)。

对列 `c_email_address` 进行加盐处理之后，就可以对其进行哈希处理，并将哈希作为 `c_email_address_pseudonym` 添加到 `gdpr.customers_lookup` 表中：

```sql
UPDATE gdpr.customers_lookup SET c_email_address_pseudonym = sha2(c_email_address,256)
```

现在，可以将此值用于所有客户密钥表。

### <a name="make-your-pseudonymization-stronger"></a><a id="make-your-pseudonymization-stronger"> </a><a id="pseudonymization-stronger"> </a>使假名化更可靠

为了减少数据库中可能混入单个盐的风险，建议在实际可行的情况下使用不同的盐（每个客户甚至每个用户使用一个盐）。  假设附加到假名标识符的数据本身不包含任何可标识个人的信息，如果删除关于哪个盐与哪个用户相关的记录并且无法重新创建记录，则其余数据应呈现为完全匿名，因此不在 GDPR 和 CCPA 的范围之内。 许多组织选择为每位用户创建多个盐，并根据业务需求定期轮换这些盐，从而在数据保护法范围之外创建完全匿名的数据。

别忘了，无论数据是“个人”数据还是“可识别”数据，都不是元素级分析，而本质上是数组级分析。  因此，虽然电子邮件地址等明显数据显然是个人数据，但本身不属于个人数据的组合也可以是个人数据。 例如，参阅 [https://aboutmyinfo.org/identity/about](https://aboutmyinfo.org/identity/about)：根据 1990 年美国人口普查局的分析，美国 87% 的人口通过邮政编码、出生日期和性别这三个属性来唯一地进行标识。  因此，决定哪些内容应作为个人标识符表的一部分或仅具有假名信息的工作表的一部分存储时，请务必考虑看似不可识别的信息的冲突本身是否可以识别。 同时确保符合自己的隐私法规，内部流程可防止尝试使用不可识别的信息（例如差分隐私、隐私保护直方图等）重新标识个人。  虽然永远不可能完全防止重新识别，但遵循这些步骤后，将会有很大的帮助。

## <a name="improve-query-performance"></a>提高查询性能

[步骤 2：执行删除 ](#gdpr-delete) 显示了如何通过提高广播阈值和 Z 排序来改进 Delta Lake 查询性能，你还应注意下面一些其他性能改进做法：

* 确保键列位于表的前 32 列之内。 Delta Lake 收集前 32 列的统计信息，这些统计信息有助于识别要删除或更新的文件。
* 使用 Azure Databricks 的 Delta Lake 中可用的自动优化功能，该功能在单独写入 Delta 表的过程中自动压缩小文件，并为主动查询的表提供显著的好处，尤其是在 Delta Lake 遇到多个小文件的情况下。 有关何时使用它的指南，请参阅[自动优化](../../delta/optimizations/auto-optimize.md)。
* 减少源表的大小（对于 `BroadcastHashJoin`）。 在确定要删除的相关数据时，这有助于 Delta Lake 利用[动态文件修剪](../../release-notes/runtime/6.1.md#dynamic-file-pruning-dfp-enabled-by-default)。 如果删除操作不在分区边界上，这将很有用。
* 对于任何修改操作（例如 `DELETE`），请尽可能具体，在 search 子句中提供所有符合条件的条件。 这样会缩小命中的文件数量，并防止事务冲突。
* 不断优化 Spark 无序、群集利用率和整个存储系统的最佳写入。

## <a name="learn-more"></a>了解详细信息

若要详细了解 Azure Databricks 上的 Delta Lake，请参阅 [Delta Lake](../../delta/index.md)。

有关由 Databricks 专家撰写的介绍使用 Delta Lake 实现 GDPR 和 CCPA 合规性的博客，请参阅：

* [如何避免在数据湖中淹没 GDPR 数据主体请求](https://databricks.com/blog/2018/05/01/how-to-avoid-drowning-in-dsrs-in-a-data-lake.html)
* [使数据湖 CCPA 符合统一的数据和分析方法](https://databricks.com/blog/2019/12/18/make-your-data-lake-ccpa-compliant.html?utm_source=bambu&utm_medium=social&utm_campaign=advocacy&blaid=321633)
* [通过 Databricks Delta 有效地更新插入到数据湖中](https://databricks.com/blog/2019/03/19/efficient-upserts-into-data-lakes-databricks-delta.html)

若要了解有关在 Azure Databricks 工作区中清除个人信息的信息，请参阅[管理工作区存储](../../administration-guide/workspace/storage.md)。