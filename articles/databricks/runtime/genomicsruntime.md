---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/29/2020
title: 用于基因组学的 Databricks Runtime - Azure Databricks
description: 了解用于基因组学的 Databricks Runtime。
toc-description: Databricks Runtime for Genomics is a variant of Databricks Runtime optimized for working with genomic and biomedical data.
ms.openlocfilehash: 24b2df1600b84109759de2a19043a39a5e4f36b0
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937754"
---
# <a name="databricks-runtime-for-genomics"></a><a id="databricks-runtime-for-genomics"> </a><a id="dbr-genomics"> </a>用于基因组学的 Databricks Runtime

用于基因组学的 Databricks Runtime（Databricks Runtime 基因组学）是为处理基因组和生物医学数据而优化的 Databricks Runtime 版本。 它是用于基因组学的 Azure Databricks 统一分析平台的组件。 若要详细了解如何开发基因组学应用程序，请参阅[基因组学](../applications/genomics/index.md)。

用于基因组学的 Databricks Runtime 已正式发布，最低版本是 6.0。

## <a name="whats-in-databricks-runtime-for-genomics"></a>用于基因组学的 Databricks Runtime 中有哪些内容？

* Databricks-Regeneron 开源库 [Glow](https://projectglow.io) 的优化版本及其所有[功能](https://glow.readthedocs.io/en/latest/)，以及：
  * 针对读取和写入变体数据的 Spark SQL 支持
  * 用于常见工作流元素的函数
  * 针对常见查询模式的优化
* 与 Apache Spark 并行化的统包管道：
  * [DNASeq](../applications/genomics/secondary/dnaseq-pipeline.md)
  * [RNASeq](../applications/genomics/secondary/rnaseq-pipeline.md)
  * [肿瘤正常排序 (MutSeq)](../applications/genomics/secondary/tumor-normal-pipeline.md)
  * [联合基因分型](../applications/genomics/tertiary/joint-genotyping-pipeline.md#joint-genotyping-pipeline)
  * [SnpeEff 变异批注](../applications/genomics/secondary/snpeff-pipeline.md#snpeff-pipeline)
* [Hail 0.2 集成](../applications/genomics/tertiary/hail.md#hail-02)
* 针对性能和可靠性进行了优化的常用开源库：
  * ADAM
  * GATK
  * Hadoop-bam
* 常用命令行工具：
  * samtools
* 参考数据（grch37 或 38，已知的 SNP 网站）

有关包含的库和版本的完整列表，请参阅[用于基因组学的 Databricks Runtime 发行说明](../release-notes/runtime/releases.md)。

## <a name="requirements"></a>要求

Azure Databricks 工作区必须已[启用](../administration-guide/clusters/genomics-runtime.md)用于基因组学的 Databricks Runtime。

## <a name="create-a-cluster-using-databricks-runtime-for-genomics"></a>使用用于基因组学的 Databricks Runtime 创建群集

[创建群集](../clusters/create.md#cluster-create)时，请从“Databricks Runtime 版本”下拉列表中选择用于基因组学的 Databricks Runtime 版本。