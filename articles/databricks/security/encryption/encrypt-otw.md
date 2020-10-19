---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/10/2020
title: 加密群集工作器节点之间的流量 - Azure Databricks
description: 了解如何在 Azure Databricks 群集工作器节点之间以在线（简称 OTW）方式加密传输中的流量。 这也称为节点间加密。
ms.openlocfilehash: 7fd7863acd59c2b74f02564e9aa5998d701bc56d
ms.sourcegitcommit: 63b9abc3d062616b35af24ddf79679381043eec1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2020
ms.locfileid: "91937812"
---
# <a name="encrypt-traffic-between-cluster-worker-nodes"></a>加密群集工作器节点之间的流量

> [!NOTE]
>
> 此功能并非适用于所有 Azure Databricks 订阅。 请联系 Microsoft 或 Databricks 客户代表，以申请访问权限。

在 Azure Databricks 的典型数据处理工作流中，通过加密的通道将用户查询或转换发送到群集。 但是，群集工作器节点之间交换的数据默认情况下未加密。 如果你的环境要求始终对数据进行加密，无论是静态加密还是传输中加密，都可以创建一个[初始化脚本](../../clusters/init-scripts.md)，该脚本将群集配置为通过 TLS 1.2 连接使用 AES 128 位加密来加密工作器节点之间的流量。

> [!NOTE]
>
> 尽管 AES 使加密例程能够利用硬件加速，但是与未加密的流量相比，仍然存在性能损失。 根据随机数据的数量，节点之间的吞吐量可能会减少，导致已加密群集上的查询花费更长的时间。

若要为工作器节点之间的流量启用加密，请创建一个用于设置 Spark 配置的[群集范围初始化脚本](../../clusters/init-scripts.md#cluster-scoped-init-script)或[全局初始化脚本](../../clusters/init-scripts.md#global-init-script)（如果希望工作区中的所有群集都使用工作器间加密）。

## <a name="get-keystore-file-and-password"></a>获取密钥存储文件和密码

将为每个工作区动态生成用于启用 SSL/HTTPS 的 JKS 密钥存储文件。 该 JKS 密钥存储文件的密码是硬编码的，不用于保护密钥存储的机密性。 不要假定密钥存储文件本身受到保护。

```bash
#!/bin/bash

keystore_file="$DB_HOME/keys/jetty_ssl_driver_keystore.jks"
keystore_password="gb1gQqZ9ZIHS"

# Use the SHA256 of the JKS keystore file as a SASL authentication secret string
sasl_secret=$(sha256sum $keystore_file | cut -d' ' -f1)

spark_defaults_conf="$DB_HOME/spark/conf/spark-defaults.conf"
driver_conf="$DB_HOME/driver/conf/config.conf"

if [ ! -e $spark_defaults_conf ] ; then
    touch $spark_defaults_conf
fi
if [ ! -e $driver_conf ] ; then
    touch $driver_conf
fi
```

## <a name="set-the-executor-configuration"></a>设置执行程序配置

```bash
# Authenticate
echo "spark.authenticate true" >> $spark_defaults_conf
echo "spark.authenticate.secret $sasl_secret" >> $spark_defaults_conf

# Configure AES encryption
echo "spark.network.crypto.enabled true" >> $spark_defaults_conf
echo "spark.network.crypto.saslFallback false" >> $spark_defaults_conf

# Configure SSL
echo "spark.ssl.enabled true" >> $spark_defaults_conf
echo "spark.ssl.keyPassword $keystore_password" >> $spark_defaults_conf
echo "spark.ssl.keyStore $keystore_file" >> $spark_defaults_conf
echo "spark.ssl.keyStorePassword $keystore_password" >> $spark_defaults_conf
echo "spark.ssl.protocol TLSv1.2" >> $spark_defaults_conf
echo "spark.ssl.standalone.enabled true" >> $spark_defaults_conf
echo "spark.ssl.ui.enabled true" >> $spark_defaults_conf
```

## <a name="set-the-driver-configuration"></a>设置驱动程序配置

```bash
head -n -a ${DB_HOME}/driver/conf/spark-branch.conf > $driver_conf

# Authenticate
echo "spark.authenticate true" >> $driver_conf
echo "spark.authenticate.secret $sasl_secret" >> $driver_conf

# Configure AES encryption
echo "spark.network.crypto.enabled true" >> $driver_conf
echo "spark.network.crypto.saslFallback false" >> $driver_conf

# Configure SSL
echo "spark.ssl.enabled true" >> $driver_conf
echo "spark.ssl.keyPassword $keystore_password" >> $driver_conf
echo "spark.ssl.keyStore $keystore_file" >> $driver_conf
echo "spark.ssl.keyStorePassword $keystore_password" >> $driver_conf
echo "spark.ssl.protocol TLSv1.2" >> $driver_conf
echo "spark.ssl.standalone.enabled true" >> $driver_conf
echo "spark.ssl.ui.enabled true" >> $driver_conf

mv $driver_conf ${DB_HOME}/driver/conf/spark-branch.conf
```

驱动程序和工作器节点的初始化完成后，这些节点之间的所有流量都将加密。