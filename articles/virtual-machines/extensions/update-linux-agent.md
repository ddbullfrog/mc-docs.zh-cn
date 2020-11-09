---
title: 从 GitHub 更新 Azure Linux 代理
description: 了解如何为 Azure 中的 Linux VM 更新 Azure Linux 代理
services: virtual-machines-linux
manager: gwallace
tags: azure-resource-manager,azure-service-management
ms.assetid: f1f19300-987d-4f29-9393-9aba866f049c
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
origin.date: 08/02/2017
author: rockboyfor
ms.date: 11/02/2020
ms.testscope: yes
ms.testdate: 11/02/2020
ms.author: v-yeche
ms.openlocfilehash: 4c3d2fb459d3c2b8aaf7f120e5422e2c2a92ed07
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93104858"
---
# <a name="how-to-update-the-azure-linux-agent-on-a-vm"></a>如何更新 VM 上的 Azure Linux 代理

若要更新 Azure 中 Linux VM 上的 [Azure Linux 代理](https://github.com/Azure/WALinuxAgent) ，则必须已具备以下条件：

- 在 Azure 中具有运行的 Linux VM。
- 使用 SHH 连接到该 Linux VM。

应始终先对 Linux 发行版存储库中的程序包进行检查。 虽然可用的程序包很有可能不是最新版本，但启用自动更新可确保 Linux 代理始终获得最新的更新。 如果从程序包管理器进行安装遇到问题，应向发行版供应商寻求支持。

> [!NOTE]
> 有关详细信息，请参阅 [Azure 认可的 Linux 分发版](../linux/endorsed-distros.md)

验证 [Azure 中的虚拟机代理的最低版本支持](https://support.microsoft.com/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support)，然后再继续。

## <a name="ubuntu"></a>Ubuntu

检查当前程序包的版本

```bash
apt list --installed | grep walinuxagent
```

更新程序包缓存

```bash
sudo apt-get -qq update
```

安装最新版本的程序包

```bash
sudo apt-get install walinuxagent
```

确保已启用自动更新。 首先，检查是否已启用自动更新：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

对于 14.04，重启 waagengt 服务

```bash
initctl restart walinuxagent
```

对于 16.04/17.04，重启 waagent 服务

```bash
systemctl restart walinuxagent.service
```

## <a name="red-hat--centos"></a>Red Hat / CentOS

<!--Not Available on ### RHEL/CentOS 6-->

## <a name="rhelcentos-7"></a>RHEL/CentOS 7

检查当前程序包的版本

```bash
sudo yum list WALinuxAgent
```

检查可用的更新

```bash
sudo yum check-update WALinuxAgent
```

安装最新版本的程序包

```bash
sudo yum install WALinuxAgent  
```

确保已启用自动更新。 首先，检查是否已启用自动更新：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

重新启动 waagent 服务

```bash
sudo systemctl restart waagent.service
```

## <a name="suse-sles"></a>SUSE SLES

### <a name="suse-sles-11-sp4"></a>SUSE SLES 11 SP4

检查当前程序包的版本

```bash
zypper info python-azure-agent
```

检查可用更新。 上面的输出将显示程序包是否为最新版。

安装最新版本的程序包

```bash
sudo zypper install python-azure-agent
```

确保已启用自动更新 

首先，检查是否已启用自动更新：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

重新启动 waagent 服务

```bash
sudo /etc/init.d/waagent restart
```

### <a name="suse-sles-12-sp2"></a>SUSE SLES 12 SP2

检查当前程序包的版本

```bash
zypper info python-azure-agent
```

检查可用的更新

上面的输出将显示程序包是否为最新版。

安装最新版本的程序包

```bash
sudo zypper install python-azure-agent
```

确保已启用自动更新 

首先，检查是否已启用自动更新：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

重新启动 waagent 服务

```bash
sudo systemctl restart waagent.service
```

## <a name="debian"></a>Debian

### <a name="debian-7-jesse-debian-7-stretch"></a>Debian 7 "Jesse"/ Debian 7 "Stretch"

检查当前程序包的版本

```bash
dpkg -l | grep waagent
```

更新程序包缓存

```bash
sudo apt-get -qq update
```

安装最新版本的程序包

```bash
sudo apt-get install waagent
```

启用代理自动更新。由于此版本的 Debian 没有 >= 2.0.16 的版本，因此 AutoUpdate 对该版本不适用。 上述命令的输出将显示程序包是否为最新版。

### <a name="debian-8-jessie--debian-9-stretch"></a>Debian 8“Jessie”/Debian 9“Stretch”

检查当前程序包的版本

```bash
apt list --installed | grep waagent
```

更新程序包缓存

```bash
sudo apt-get -qq update
```

安装最新版本的程序包

```bash
sudo apt-get install waagent
```

请确保先启用自动更新。检查它是否已启用：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
Restart the waagent service
sudo systemctl restart walinuxagent.service
```

<!-- Not Available on ## Oracle Linux 6 and Oracle Linux 7 -->

## <a name="update-the-linux-agent-when-no-agent-package-exists-for-distribution"></a>分发不存在代理程序包时，请更新 Linux 代理

通过在命令行上键入 `sudo yum install wget` 来安装 wget（某些发行版，例如 CentOS，未在默认情况下安装它，）。

<!--MOONCAKE CUSTOMIZATION ON THE ABOVE SEVTENCE-->
<!-- Not Available on Red Hat, and Oracle -->

### <a name="1-download-the-latest-version"></a>1.下载最新版本
在网页中打开 [GitHub 中的 Azure Linux 代理版本](https://github.com/Azure/WALinuxAgent/releases)，并找到最新的版本号。 （可以通过键入 `waagent --version` 查明当前版本。）

对于 2.2.x 或更高版本，请键入：
```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.x.zip
unzip v2.2.x.zip
cd WALinuxAgent-2.2.x
```

以下行使用版本 2.2.0 作为示例：

```bash
wget https://github.com/Azure/WALinuxAgent/archive/v2.2.14.zip
unzip v2.2.14.zip  
cd WALinuxAgent-2.2.14
```

### <a name="2-install-the-azure-linux-agent"></a>2.安装 Azure Linux 代理

对于版本 2.2.x，请使用：可能需要先安装程序包 `setuptools` -- 详情请参阅 [此处](https://pypi.python.org/pypi/setuptools)。 然后运行：

```bash
sudo python setup.py install
```

确保已启用自动更新。 首先，检查是否已启用自动更新：

```bash
cat /etc/waagent.conf
```

查找“AutoUpdate.Enabled”。 如果看到以下输出，则表示已启用：

```bash
# AutoUpdate.Enabled=y
AutoUpdate.Enabled=y
```

若要允许运行：

```bash
sudo sed -i 's/# AutoUpdate.Enabled=n/AutoUpdate.Enabled=y/g' /etc/waagent.conf
```

### <a name="3-restart-the-waagent-service"></a>3.重新启动 waagent 服务
对于大多数 linux 发行版：

```bash
sudo service waagent restart
```

对于 Ubuntu，使用：

```bash
sudo service walinuxagent restart
```

对于 CoreOS，使用：

```bash
sudo systemctl restart waagent
```

### <a name="4-confirm-the-azure-linux-agent-version"></a>4.确认 Azure Linux 代理版本

```bash
waagent -version
```

对于 CoreOS，上面的命令可能无效。

会看到 Linux 代理版本已更新为新版本。

有关 Azure Linux 代理的详细信息，请参阅 [Azure Linux 代理自述文件](https://github.com/Azure/WALinuxAgent)。

<!-- Update_Description: update meta properties, wording update, update link -->