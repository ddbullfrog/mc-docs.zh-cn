---
title: Azure 网络观察程序的数据驻留 | Azure
description: 本文介绍 Azure 网络观察程序服务的数据驻留。
services: network-watcher
documentationcenter: na
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 07/20/2020
author: rockboyfor
ms.date: 10/19/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 120e5fd8d7e089509f210231de9ee0931789747c
ms.sourcegitcommit: 7320277f4d3c63c0b1ae31ba047e31bf2fe26bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92118804"
---
<!--Verified successfully for charactors only-->
# <a name="data-residency-for-azure-network-watcher"></a>Azure 网络观察程序的数据驻留
Azure 网络观察程序不会存储客户数据，连接监视器（预览版）服务除外。

## <a name="connection-monitor-preview-data-residency"></a>连接监视器（预览版）数据驻留
连接监视器（预览版）服务会存储客户数据。 这些数据由网络观察程序自动存储在单个区域中。 因此，连接监视器（预览版）会自动满足区域内数据驻留要求，包括在[信任中心](https://azuredatacentermap.azurewebsites.net/)指定的要求。

<!--Not Available on ## Singapore data residency-->


## <a name="next-steps"></a>后续步骤

* 阅读[网络观察程序](/network-watcher/network-watcher-monitoring-overview)的概述。

<!-- Update_Description: new article about data residency -->
<!--NEW.date: 10/19/2020-->