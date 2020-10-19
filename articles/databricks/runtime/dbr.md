---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/23/2020
title: Databricks Runtime - Azure Databricks
description: 了解基础 Databricks 运行时 Databricks Runtime。
toc-description: Databricks Runtime includes Apache Spark but also adds a number of components and updates that substantially improve the usability, performance, and security of big data analytics.
ms.openlocfilehash: 69ab2e1e819c7b8d3046fecfb44a6cb438737c76
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937752"
---
# <a name="databricks-runtime"></a><a id="databricks-runtime"> </a><a id="dbr"> </a>Databricks Runtime

Databricks Runtime 包括 Apache Spark，但还添加了许多可以显著提高大数据分析可用性、性能和安全性的组件与更新：

* [Delta Lake](../delta/index.md) 是基于 Apache Spark 构建的下一代存储层，可提供 ACID 事务、优化的布局和索引以及针对数据管道生成的执行引擎改进。
* 已安装的 Java、Scala、Python 和 R 库
* Ubuntu 及其随附的系统库
* 启用了 GPU 的群集的 GPU 库
* 与平台的其他组件（如笔记本、作业和群集管理器）集成的 Databricks 服务

有关每个运行时版本的内容的信息，请参阅[发行说明](../release-notes/runtime/releases.md)。

## <a name="runtime-versioning"></a>运行时版本控制

将定期发布 Databricks Runtime 版本：

* 主版本的变化通过递增小数点之前的版本号来表示（例如从 3.5 跳转到 4.0）。 它们在发生重大更改时发布，其中一些可能无法向后兼容。
* 功能版本的变化通过递增小数点之后的版本号来表示（例如从 3.4 跳转到 3.5）。 每个主要版本都包含多个功能版。 功能版总是向后兼容其主要版本中的先前版本。
* 长期支持版本由 LTS 限定符（例如 3.5 LTS）表示  。 对于每个主要版本，我们都声明一个“规范”功能版本，并为其提供为期两年的支持。 有关详细信息，请参阅 [Databricks 运行时支持生命周期](../release-notes/runtime/databricks-runtime-ver.md#runtime-support)。