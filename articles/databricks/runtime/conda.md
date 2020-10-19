---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 05/05/2020
title: 带有 Conda 的 Databricks Runtime - Azure Databricks
description: 带有 Conda 的 Databricks Runtime 是基于 Conda 的 Databricks Runtime 试验版本，它不再可用。 请改为用于机器学习的 Databricks Runtime。
toc-description: Databricks Runtime with Conda is a variant of Databricks Runtime that provides an optimized list of default packages and a flexible Python environment that enables maximum control over packages.
ms.openlocfilehash: ec9fafc78d77be4b32bab842210637f8c2998fe8
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937755"
---
# <a name="databricks-runtime-with-conda"></a><a id="condaruntime"> </a><a id="databricks-runtime-with-conda"> </a>带有 Conda 的 Databricks Runtime

带有 Conda 的 Databricks Runtime 是基于 [Conda](https://docs.conda.io/en/latest/) 环境（而非 Python 虚拟环境）的 Databricks Runtime，仅在 beta 版本中可用。 如果要使用 Conda 来管理 Python 库和环境，请使用 [用于机器学习的 Databricks Runtime](mlruntime.md) 的[受支持版本](../release-notes/runtime/releases.md#supported-releases)。