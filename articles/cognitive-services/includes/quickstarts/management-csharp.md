---
title: 快速入门：适用于 .NET 的 Azure 管理客户端库
description: 在本快速入门中，开始使用适用于 .NET 的 Azure 管理客户端库。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: 9af3800c180af1a3f9f8aa2f5df80ebc8d2f166a
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106479"
---
[参考文档](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/management?view=azure-dotnet) | [库源代码](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices) | [包 (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.Management.CognitiveServices/) | [示例](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Microsoft.Azure.Management.CognitiveServices/tests)

## <a name="prerequisites"></a>先决条件

* 有效的 Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。
* [.NET Core](https://dotnet.microsoft.com/download/dotnet-core) 的当前版本。

[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-c-application"></a>新建 C# 应用程序

创建新的 .NET Core 应用程序。 在控制台窗口（例如 cmd、PowerShell 或 Bash）中，使用 `dotnet new` 命令创建名为 `azure-management-quickstart` 的新控制台应用。 此命令将创建包含单个源文件的简单“Hello World”C# 项目： *program.cs* 。 

```console
dotnet new console -n azure-management-quickstart
```

将目录更改为新创建的应用文件夹。 可使用以下代码生成应用程序：

```console
dotnet build
```

生成输出不应包含警告或错误。 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>安装客户端库

在应用程序目录中，使用以下命令安装适用于 .NET 的 Azure 管理客户端库：

```console
dotnet add package Microsoft.Azure.Management.CognitiveServices
dotnet add package Microsoft.Azure.Management.Fluent
dotnet add package Microsoft.Azure.Management.ResourceManager.Fluent
```

如果你使用的是 Visual Studio IDE，客户端库可用作可下载的 NuGet 包。

### <a name="import-libraries"></a>导入库

打开“program.cs”并将以下 `using` 语句添加到文件顶部：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="authenticate-the-client"></a>验证客户端

将以下字段添加到“program.cs”的根目录，并使用创建的服务主体和 Azure 帐户信息填写其值。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

然后，在 Main 方法中，使用这些值构造 CognitiveServicesManagementClient 对象。 所有 Azure 管理操作都需要此对象。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="call-management-methods"></a>调用管理方法

将以下代码添加到 Main 方法中，以列出可用资源、创建示例资源、列出拥有的资源，然后删除示例资源。 你将在后续步骤中定义这些方法。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

### <a name="choose-a-service-and-pricing-tier"></a>选择服务和定价层

创建新资源时，需要知道要使用的服务的种类，以及所需的[定价层](https://www.azure.cn/pricing/details/cognitive-services/)（或 SKU）。 创建资源时，将此信息和其他信息用作参数。 可以通过在脚本中调用以下方法来获取可用认知服务种类的列表：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

若要创建并订阅新的认知服务资源，请使用 Create 方法。 此方法向传入的资源组添加新的可计费资源。 创建新资源时，需要知道要使用的服务的种类，以及其定价层（或 SKU）和 Azure 位置。 下面的方法使用所有这些参数并创建资源。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="view-your-resources"></a>查看资源

若要查看 Azure 帐户下的所有资源（跨所有资源组），请使用以下方法：

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="delete-a-resource"></a>删除资源

下面的方法从给定的资源组中删除指定的资源。

```csharp
// <snippet_using>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent.Authentication;
using Microsoft.Azure.Management.CognitiveServices;
using Microsoft.Azure.Management.CognitiveServices.Models;
// </snippet_using>

/* Note: Install the following NuGet packages:
Microsoft.Azure.Management.CognitiveServices 
Microsoft.Azure.Management.Fluent
Microsoft.Azure.Management.ResourceManager.Fluent
*/

namespace ConsoleApp1
{
    class Program
    {
        /*
        The application ID and secret of the service principal you are using to connect to the Azure Management Service.

        To create a service principal with the Azure CLI, see:
        /cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest
        To install the Azure CLI, see:
        /cli/install-azure-cli?view=azure-cli-latest

        To create a service principal with Azure PowerShell, see: 
        https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps?view=azps-3.3.0
        To install Azure PowerShell, see:
        https://github.com/Azure/azure-powershell

        When you create a service principal, you will see it has both an ID and an application ID.
        For example, if you create a service principal with Azure PowerShell, it should look like the following:

        Secret                : System.Security.SecureString
        ServicePrincipalNames : {<application ID>, <application URL>}
        ApplicationId         : <application ID>
        ObjectType            : ServicePrincipal
        DisplayName           : <name>
        Id                    : <ID>
        Type                  :

        Be sure to use the service principal application ID, not simply the ID. 
        */

        // <snippet_constants>
        const string  service_principal_application_id = "TODO_REPLACE";
        const string  service_principal_secret = "TODO_REPLACE";

        /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
        const string  subscription_id = "TODO_REPLACE";

        /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
        const string  tenant_id = "TODO_REPLACE";

        /* The name of the Azure resource group in which you want to create the resource.
        You can find resource groups in the Azure Dashboard under Home > Resource groups. */
        const string  resource_group_name = "TODO_REPLACE";
        // </snippet_constants>

        // <snippet_list_avail>
        static void list_available_kinds_skus_locations(CognitiveServicesManagementClient client)
        {

            Console.WriteLine("Available SKUs:");
            var result = client.ResourceSkus.List();
            Console.WriteLine("Kind\tSKU Name\tSKU Tier\tLocations");
            foreach (var x in result) {
                var locations = "";
                foreach (var region in x.Locations)
                {
                    locations += region;
                }
                Console.WriteLine(x.Kind + "\t" + x.Name + "\t" + x.Tier + "\t" + locations);
            };
        }
        // </snippet_list_avail>

        // <snippet_list>
        static void list_resources(CognitiveServicesManagementClient client)
        {
            Console.WriteLine("Resources in resource group: " + resource_group_name);
            var result = client.Accounts.ListByResourceGroup(resource_group_name);
            foreach (var x in result)
            {
                Console.WriteLine("ID: " + x.Id);
                Console.WriteLine("Name: " + x.Name);
                Console.WriteLine("Type: " + x.Type);
                Console.WriteLine("Kind: " + x.Kind);
                Console.WriteLine();
            }
        }
        // </snippet_list>

        // <snippet_create>
        static void create_resource(CognitiveServicesManagementClient client, string resource_name, string kind, string account_tier, string location)
        {
            Console.WriteLine("Creating resource: " + resource_name + "...");
            // The parameter "properties" must be an empty object.
            CognitiveServicesAccount parameters = 
                new CognitiveServicesAccount(null, null, kind, location, resource_name, new CognitiveServicesAccountProperties(), new Sku(account_tier));
            var result = client.Accounts.Create(resource_group_name, account_tier, parameters);
            Console.WriteLine("Resource created.");
            Console.WriteLine("ID: " + result.Id);
            Console.WriteLine("Kind: " + result.Kind);
            Console.WriteLine();
        }
        // </snippet_create>

        // <snippet_delete>
        static void delete_resource(CognitiveServicesManagementClient client, string resource_name)
        {
            Console.WriteLine("Deleting resource: " + resource_name + "...");
            client.Accounts.Delete (resource_group_name, resource_name);

            Console.WriteLine("Resource deleted.");
            Console.WriteLine();
        }
        // </snippet_delete>

    static void Main(string[] args)
        {
            /* For more information about authenticating, see:
             * https://docs.microsoft.com/dotnet/azure/dotnet-sdk-azure-authenticate?view=azure-dotnet
             */

            // <snippet_assigns>
            var service_principal_credentials = new ServicePrincipalLoginInformation ();
            service_principal_credentials.ClientId = service_principal_application_id;
            service_principal_credentials.ClientSecret = service_principal_secret;

            var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(service_principal_application_id, service_principal_secret, tenant_id, AzureEnvironment.AzureGlobalCloud);
            var client = new CognitiveServicesManagementClient(credentials);
            client.SubscriptionId = subscription_id;
            // </snippet_assigns>

            // <snippet_calls>
            // List all available resource kinds, SKUs, and locations for your Azure account:
            list_available_kinds_skus_locations(client);
        
            // Create a resource with kind TextTranslation, F0 (free tier), location global.
            create_resource(client, "test_resource", "TextTranslation", "F0", "Global");

            // List all resources for your Azure account:
            list_resources(client);

            // Delete the resource.
            delete_resource(client, "test_resource");

            Console.WriteLine("Press any key to exit.");
            Console.ReadKey();
            // </snippet_calls>
        }
    }
}
```

## <a name="run-the-application"></a>运行应用程序

从应用程序目录使用 `dotnet run` 命令运行应用程序。

```dotnet
dotnet run
```

## <a name="see-also"></a>另请参阅

* [Azure 管理 SDK 参考文档](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/management?view=azure-dotnet)
* [什么是 Azure 认知服务？](../../Welcome.md)
* [对 Azure 认知服务的请求进行身份验证](../../authentication.md)
* [使用 Azure 门户创建新资源](../../cognitive-services-apis-create-account.md)

