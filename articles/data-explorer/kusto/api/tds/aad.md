---
title: 将 MS-TDS 与 Azure Active Directory 配合使用 - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍如何在 Azure 数据资源管理器中将 MS-TDS 与 Azure Active Directory 配合使用。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
origin.date: 01/02/2019
ms.date: 10/29/2020
ms.openlocfilehash: a47f4ca5cc64c9d3517babf62b6286f72b91e2bf
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93103897"
---
# <a name="ms-tds-with-azure-active-directory"></a>将 MS-TDS 与 Azure Active Directory 配合使用

## <a name="azure-ad-user-authentication"></a>Azure AD 用户身份验证

支持 Azure AD 用户身份验证的 SQL 客户端可以与 Azure 数据资源管理器配合使用。

### <a name="net-sql-client-user"></a>.NET SQL 客户端（用户）

例如，对于集成 Azure AD：
```csharp
    var csb = new SqlConnectionStringBuilder()
    {
        InitialCatalog = "mydatabase",
        Authentication = SqlAuthenticationMethod.ActiveDirectoryIntegrated,
        DataSource = "mykusto.kusto.chinacloudapi.cn"
    };
```

Kusto 支持使用已获取的访问令牌进行身份验证：
```csharp
    var csb = new SqlConnectionStringBuilder()
    {
        InitialCatalog = "mydatabase",
        DataSource = "mykusto.kusto.chinacloudapi.cn"
    };
    using (var connection = new SqlConnection(csb.ToString()))
    {
        connection.AccessToken = accessToken;
        await connection.OpenAsync();
    }
```

### <a name="jdbc-user"></a>JDBC（用户）

```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.Statement;
import com.microsoft.sqlserver.jdbc.SQLServerDataSource;
import com.microsoft.aad.adal4j.*;

public class Sample {
  public static void main(String[] args) throws Exception {
    AuthenticationResult authenticationResult = futureAuthenticationResult.get();
    SQLServerDataSource ds = new SQLServerDataSource();
    ds.setServerName("<your cluster DNS name>");
    ds.setDatabaseName("<your database name>");
    ds.setHostNameInCertificate("*.kusto.chinacloudapi.cn"); // Or appropriate regional domain.
    ds.setAuthentication("ActiveDirectoryIntegrated");
    try (Connection connection = ds.getConnection();
         Statement stmt = connection.createStatement();) {
      ResultSet rs = stmt.executeQuery("<your T-SQL query>");
      /*
      Read query result.
      */
    } catch (Exception e) {
      System.out.println();
      e.printStackTrace();
    }
  }
}
```

## <a name="azure-ad-application-authentication"></a>Azure AD 应用程序身份验证

为 Kusto 预配的 Azure AD 应用程序可使用支持 Azure AD 的 SQL 客户端库连接到 Kusto。

### <a name="net-sql-client-application"></a>.NET SQL 客户端（应用程序）

在假设你已为 Azure AD 应用程序预配了 ApplicationClientId 和 ApplicationKey，并授予其访问群集 ClusterDnsName 上的数据库 DatabaseName 的权限的情况下，以下示例演示如何使用 .NET SQL 客户端从此 Azure AD 应用程序进行查询。

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System;
using System.Data;
using System.Data.SqlClient;

namespace Sample
{
  class Program
  {
    private static async Task<string> ObtainToken()
    {
      var authContext = new AuthenticationContext(
        // Can also use tenant ID.
        "https://login.partner.microsoftonline.cn/<your AAD tenant name>");
      var applicationCredentials = new ClientCredential(
        "<your application client ID>",
        "<your application key>");
      var result = await authContext.AcquireTokenAsync(
        "https://<your cluster DNS name>",
        applicationCredentials);
      return result.AccessToken;
    }

    private static async Task QuerySample()
    {
      var csb = new SqlConnectionStringBuilder()
      {
        InitialCatalog = "<your database name>",
        DataSource = "<your cluster DNS name>"
      };
      using (var connection = new SqlConnection(csb.ToString()))
      {
        connection.AccessToken = await ObtainToken();
        await connection.OpenAsync();
        using (var command = new SqlCommand(
          "<your T-SQL query>",
          connection))
        {
          var reader = await command.ExecuteReaderAsync();
          /*
          Read query result.
          */
        }
      }
    }
  }
}
```

### <a name="jdbc-application"></a>JDBC（应用程序）

```java
import java.sql.*;
import com.microsoft.sqlserver.jdbc.*;
import com.microsoft.aad.adal4j.*;

public class Sample {
  public static void main(String[] args) throws Throwable {
    ExecutorService service = Executors.newFixedThreadPool(1);
    // Can also use tenant name.
    String url = "https://login.partner.microsoftonline.cn/<your AAD tenant ID>";
    AuthenticationContext authenticationContext =
      new AuthenticationContext(url, false, service);
    ClientCredential  clientCredential = new ClientCredential(
      "<your application client ID>",
      "<your application key>");
    Future<AuthenticationResult> futureAuthenticationResult =
      authenticationContext.acquireToken(
        "https://<your cluster DNS name>",
        clientCredential,
        null);
    AuthenticationResult authenticationResult = futureAuthenticationResult.get();
    SQLServerDataSource ds = new SQLServerDataSource();
    ds.setServerName("<your cluster DNS name>");
    ds.setDatabaseName("<your database name>");
    ds.setAccessToken(authenticationResult.getAccessToken());
    connection = ds.getConnection();
    statement = connection.createStatement();
    ResultSet rs = statement.executeQuery("<your T-SQL query>");
    /*
    Read query result.
    */
  }
}
```