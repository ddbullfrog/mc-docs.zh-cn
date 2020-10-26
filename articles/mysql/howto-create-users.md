---
title: 创建数据库和用户 - Azure Database for MySQL
description: 本文介绍如何创建新的用户帐户，以与 Azure Database for MySQL 服务器进行交互。
author: WenJason
ms.author: v-jay
ms.service: mysql
ms.topic: how-to
origin.date: 4/2/2020
ms.date: 10/19/2020
ms.openlocfilehash: 1aac4fba2499aacbd377552e8e796c917e4c385c
ms.sourcegitcommit: ba01e2d1882c85ebeffef344ef57afaa604b53a0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2020
ms.locfileid: "92041787"
---
# <a name="create-databases-and-users-in-azure-database-for-mysql-server"></a>在 Azure Database for MySQL 服务器中创建数据库和用户

> [!NOTE]
> 将要查看的是 Azure Database for MySQL 的新服务。 若要查看经典 MySQL Database for Azure 的文档，请访问[此页](https://docs.azure.cn/zh-cn/mysql-database-on-azure/)。

本文介绍如何在 Azure Database for MySQL 服务器中创建用户。

首次创建 Azure Database for MySQL 时，需要提供服务器管理员登录用户名和密码。 有关详细信息，可以参考[快速入门](quickstart-create-mysql-server-database-using-azure-portal.md)。 你可以从 Azure 门户中找到你的服务器管理员登录用户名。

服务器管理员用户可获得服务器的某些权限，如下所示： 

   SELECT、INSERT、UPDATE、DELETE、CREATE、DROP、RELOAD、PROCESS、REFERENCES、INDEX、ALTER、SHOW DATABASES、CREATE TEMPORARY TABLES、LOCK TABLES、EXECUTE、REPLICATION SLAVE、REPLICATION CLIENT、CREATE VIEW、SHOW VIEW、CREATE ROUTINE、ALTER ROUTINE、CREATE USER、EVENT、TRIGGER


创建 Azure Database for MySQL 服务器后，你可以使用第一个服务器管理员用户帐户来创建其他用户，并授予这些用户管理员访问权限。 此外，服务器管理员帐户还可以用于创建只能访问各个数据库架构的权限较低的用户。

> [!NOTE]
> 不支持 SUPER 权限和 DBA 角色。 请在“限制”一文中查看[权限](concepts-limits.md#privilege-support)，以了解服务中不支持的权限。

## <a name="how-to-create-database-with-non-admin-user-in-azure-database-for-mysql"></a>如何在 Azure Database for MySQL 中使用非管理员用户创建数据库

1. 获取连接信息和管理员用户名。
   若要连接到数据库服务器，需提供完整的服务器名称和管理员登录凭据。 你可以在 Azure 门户的服务器“概述”页或“属性”页中轻松找到服务器名称和登录信息。  

2. 使用管理员帐户和密码连接到你的数据库服务器。 使用你的首选客户端工具，如 MySQL Workbench、mysql.exe、HeidiSQL 或其他工具。
   如果你不确定如何连接，请参阅[如何使用 MySQL Workbench 连接和查询单一服务器的数据](./connect-workbench.md)

3. 编辑并运行下面的 SQL 代码。 将占位符值 `db_user` 替换为预期的新用户名，并将占位符值 `testdb` 替换为你自己的数据库名称。

   出于举例的目的，此 sql 代码语法将创建一个名为 testdb 的新数据库。 然后，它在 MySQL 服务中创建新用户，并将所有权限授予该用户的新数据库架构 (testdb.\*)。

   ```sql
   CREATE DATABASE testdb;

   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';

   FLUSH PRIVILEGES;
   ```

4. 验证数据库中的授予。

   ```sql
   USE testdb;

   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. 使用新用户名和密码登录到服务器，指定选定的数据库。 此示例显示了 mysql 命令行。 使用此命令，会提示你输入用户名的密码。 替换你自己的服务器名称、数据库名称和用户名。

   ```azurecli
   mysql --host mydemoserver.mysql.database.chinacloudapi.cn --database testdb --user db_user@mydemoserver -p
   ```

## <a name="how-to-create-additional-admin-users-in-azure-database-for-mysql"></a>如何在 Azure Database for MySQL 中创建其他管理员用户

1. 获取连接信息和管理员用户名。
   若要连接到数据库服务器，需提供完整的服务器名称和管理员登录凭据。 你可以在 Azure 门户的服务器“概述”页或“属性”页中轻松找到服务器名称和登录信息。  

2. 使用管理员帐户和密码连接到你的数据库服务器。 使用你的首选客户端工具，如 MySQL Workbench、mysql.exe、HeidiSQL 或其他工具。
   如果你不确定如何连接，请参阅[使用 MySQL Workbench 连接和查询数据](./connect-workbench.md)

3. 编辑并运行下面的 SQL 代码。 将占位符值 `new_master_user` 替换为你的新用户名。 此语法会将所有数据库架构 ( *.* ) 上列出的权限授予该用户名（本示例中的 new_master_user）。

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION;

   FLUSH PRIVILEGES;
   ```

4. 验证授予

   ```sql
   USE sys;

   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="next-steps"></a>后续步骤

针对新用户计算机的 IP 地址打开防火墙，使其能够连接：
- [在单一服务器上创建和管理防火墙规则](howto-manage-firewall-using-portal.md) 

有关用户帐户管理的详细信息，请参阅 MySQL 产品文档，了解[用户帐户管理](https://dev.mysql.com/doc/refman/5.7/en/access-control.html)、[GRANT 语法](https://dev.mysql.com/doc/refman/5.7/en/grant.html)和[权限](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html)。
