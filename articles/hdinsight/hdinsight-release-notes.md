---
title: Azure HDInsight 发行说明
description: Azure HDInsight 的最新发行说明。 获取 Hadoop、Spark、R Server、Hive 和更多工具的开发技巧和详细信息。
services: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
origin.date: 10/07/2020
ms.date: 11/02/2020
ms.openlocfilehash: 40d680495dc5e68f21956d9c045ccd64d2b699d6
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472612"
---
# <a name="release-notes"></a>发行说明

本文提供有关 **最新** Azure HDInsight 版本更新的信息。 有关较早版本的信息，请参阅 [HDInsight 发行说明存档](hdinsight-release-notes-archive.md)。

## <a name="summary"></a>摘要

Azure HDInsight 是 Azure 中最受企业客户青睐的开源分析服务之一。

## <a name="release-date-10082020"></a>发行日期：2020/10/08

此版本适用于 HDInsight 3.6 和 HDInsight 4.0。 HDInsight 发行版在几天后即会在所有区域中推出。 此处的发行日期是指在第一个区域中的发行日期。 如果看不到以下更改，请耐心等待，几天后发行版会在你所在的区域推出。

## <a name="new-features"></a>新增功能
### <a name="hdinsight-private-clusters-with-no-public-ip-and-private-link-preview"></a>没有公共 IP 和专用链接的 HDInsight 专用群集（预览版）
HDInsight 现支持创建没有公共 IP 和专用链接（用于访问相应群集）的群集（处于预览状态）。 客户可以使用新的高级网络设置来创建没有公共 IP 的完全独立的群集，并可以使用自己的专用终结点来访问该群集。 

### <a name="moving-to-azure-virtual-machine-scale-sets"></a>迁移到 Azure 虚拟机规模集
HDInsight 目前使用 Azure 虚拟机来预配群集。 从此版本开始，该服务将逐渐迁移到 [Azure 虚拟机规模集](/virtual-machine-scale-sets/overview)。 整个过程可能需要几个月。 迁移区域和订阅后，新创建的 HDInsight 群集将在虚拟机规模集上运行，而无需客户执行任何操作。 预计不会有中断性变更。

## <a name="deprecation"></a>弃用
#### <a name="deprecation-of-hdinsight-36-ml-services-cluster"></a>弃用 HDInsight 3.6 ML 服务群集
HDInsight 3.6 ML 服务群集类型将于 2020 年 12 月 31 日终止支持。 之后，客户将不会创建新的 3.6 ML 服务群集。 现有群集将在没有 Microsoft 支持的情况下按原样运行。 请在[此处](/hdinsight/hdinsight-component-versioning#available-versions)检查 HDInsight 版本的有效期限和群集类型。

## <a name="behavior-changes"></a>行为更改
此版本没有行为变更。

## <a name="upcoming-changes"></a>即将推出的更改
即将发布的版本中将推出以下变更。

### <a name="ability-to-select-different-zookeeper-virtual-machine-sizes-for-spark-hadoop-and-ml-services"></a>能够为 Spark、Hadoop 和 ML 服务选择不同的 Zookeeper 虚拟机大小
目前，HDInsight 不支持为 Spark、Hadoop 和 ML 服务群集类型自定义 Zookeeper 节点大小。 默认情况下为 A2_v2/A2 虚拟机大小（免费提供）。 在即将发布的版本中，可以选择最适合自己方案的 Zookeeper 虚拟机大小。 虚拟机大小不是 A2_v2/A2 的 Zookeeper 节点需要付费。 A2_v2 和 A2 虚拟机仍免费提供。

## <a name="bug-fixes"></a>Bug 修复
HDInsight 会持续改善群集的可靠性和性能。 

## <a name="component-version-change"></a>组件版本更改
此发行版未发生组件版本更改。 可以在[此文档](/hdinsight/hdinsight-component-versioning#apache-hadoop-components-available-with-different-hdinsight-versions)中查找 HDInsight 4.0 和 HDInsight 3.6 的当前组件版本。
