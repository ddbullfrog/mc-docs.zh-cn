---
title: 查看 Azure 导入/导出作业状态 - v1 | Microsoft Docs
description: 了解如何使用导入或导出作业创建的日志文件查看作业的状态。
author: WenJason
services: storage
ms.service: storage
ms.topic: how-to
origin.date: 01/26/2017
ms.date: 09/28/2020
ms.author: v-jay
ms.subservice: common
ms.openlocfilehash: e75928282734eedb812d18048444fa9fffd85d28
ms.sourcegitcommit: 119a3fc5ffa4768b1bd8202191091bd4d873efb4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2020
ms.locfileid: "91026643"
---
# <a name="reviewing-azure-importexport-job-status-with-copy-log-files"></a>使用复制日志文件查看 Azure 导入/导出作业状态
当 Azure 导入/导出服务处理与导入或导出作业关联的驱动器时，它会将副本日志文件写入到用于导入或导出 blob 的存储帐户。 该日志文件包含与已导入或导出的每个文件相关的详细状态。 当你查询已完成作业的状态时，该服务将返回每个副本日志文件的 URL。 有关详细信息，请参阅[获取作业](https://docs.microsoft.com/rest/api/storageimportexport/Jobs/Get)。  

## <a name="example-urls"></a>示例 URL

下面是包含两个驱动器的导入作业的复制日志文件的示例 URL：  

 `http://myaccount.blob.core.chinacloudapi.cn/ImportExportStatesPath/waies/myjob_9WM35C2V_20130921-034307-902_error.xml`  

 `http://myaccount.blob.core.chinacloudapi.cn/ImportExportStatesPath/waies/myjob_9WM45A6Q_20130921-042122-021_error.xml`  

 有关复制日志的格式以及状态代码的完整列表，请参阅[导入/导出服务日志文件格式](../storage-import-export-file-format-log.md)。  

## <a name="next-steps"></a>后续步骤

 * [设置 Azure 导入/导出工具](storage-import-export-tool-setup-v1.md)   
 * [为导入作业准备硬盘驱动器](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
 * [修复导入作业](../storage-import-export-tool-repairing-an-import-job-v1.md)   
 * [修复导出作业](../storage-import-export-tool-repairing-an-export-job-v1.md)
