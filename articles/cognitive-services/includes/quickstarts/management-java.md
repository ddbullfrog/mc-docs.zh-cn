---
title: 快速入门：适用于 Java 的 Azure 管理客户端库
description: 在本快速入门中，开始使用适用于 Java 的 Azure 管理客户端库。
services: cognitive-services
author: Johnnytechn
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2020
ms.author: v-johya
ms.openlocfilehash: d2086f6779149b5620fd0133723cc9242e9dd16b
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106475"
---
[参考文档](https://docs.microsoft.com/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable) | [库源代码](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/mgmt-v2017_04_18/src/main/java/com/microsoft/azure/management/cognitiveservices/v2017_04_18) | [包 (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure/azure-mgmt-cognitiveservices)

## <a name="prerequisites"></a>先决条件

* 有效的 Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。
* 最新版本的 [Java 开发工具包 (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Gradle 生成工具](https://gradle.org/install/)，或其他依赖项管理器。


[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-java-application"></a>创建新的 Java 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。 

```console
mkdir myapp && cd myapp
```

从工作目录运行 `gradle init` 命令。 此命令将创建 Gradle 的基本生成文件，包括 *build.gradle.kts* ，在运行时将使用该文件创建并配置应用程序。

```console
gradle init --type basic
```

当提示你选择一个 **DSL** 时，选择 **Kotlin** 。

在工作目录中运行以下命令：

```console
mkdir -p src/main/java
```

### <a name="install-the-client-library"></a>安装客户端库

本快速入门使用 Gradle 依赖项管理器。 可以在 [Maven 中央存储库](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer)中找到客户端库以及其他依赖项管理器的信息。

在项目的 build.gradle.kts 文件中，以 `implementation` 语句的形式包含客户端库及所需的插件和设置。

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.microsoft.azure", name = "azure-mgmt-cognitiveservices", version = "1.10.0-beta")
}
```

### <a name="import-libraries"></a>导入库

导航到新的 src/main/java 文件夹，并创建名为 Management.java 的文件。 在喜好的编辑器或 IDE 中打开该文件并添加以下 `import` 语句：

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="authenticate-the-client"></a>验证客户端

在 Management.java 中添加类，然后在其中添加以下字段及其值。 使用创建的服务主体和其他 Azure 帐户信息填写其值。

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

然后，在 main 方法中，使用这些值构造 CognitiveServicesManager 对象 。 所有 Azure 管理操作都需要此对象。

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="call-management-methods"></a>调用管理方法

将以下代码添加到 Main 方法中，以列出可用资源、创建示例资源、列出拥有的资源，然后删除示例资源。 你将在后续步骤中定义这些方法。

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

### <a name="choose-a-service-and-pricing-tier"></a>选择服务和定价层

创建新资源时，需要知道要使用的服务的种类，以及所需的[定价层](https://www.azure.cn/pricing/details/cognitive-services/)（或 SKU）。 创建资源时，将此信息和其他信息用作参数。 可以通过调用以下方法来获取可用认知服务“种类”的列表：

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="create-a-cognitive-services-resource"></a>创建认知服务资源

若要创建并订阅新的认知服务资源，请使用 create 方法。 此方法向传入的资源组添加新的可计费资源。 创建新资源时，需要知道要使用的服务的种类，以及其定价层（或 SKU）和 Azure 位置。 下面的方法使用所有这些参数并创建资源。

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="view-your-resources"></a>查看资源

若要查看 Azure 帐户下的所有资源（跨所有资源组），请使用以下方法：

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="delete-a-resource"></a>删除资源

下面的方法从给定的资源组中删除指定的资源。

```java
// <snippet_imports>
import com.microsoft.azure.*;
import com.microsoft.azure.arm.resources.Region;
import com.microsoft.azure.credentials.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.*;
import com.microsoft.azure.management.cognitiveservices.v2017_04_18.implementation.*;

import java.io.*;
import java.lang.Object.*;
import java.util.*;
import java.net.*;
// </snippet_imports>

/* To compile and run, enter the following at a command prompt:
 * javac Quickstart.java -cp .;lib\*
 * java -cp .;lib\* Quickstart
 * This presumes your libraries are stored in a folder named "lib"
 * directly under the current folder. If not, please adjust the
 * -classpath/-cp value accordingly.
 */

public class Quickstart {
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
    */

    // <snippet_constants>
    /*
    Be sure to use the service pricipal application ID, not simply the ID. 
    */
    private static String applicationId = "INSERT APPLICATION ID HERE";
    private static String applicationSecret = "INSERT APPLICATION SECRET HERE";

    /* The ID of your Azure subscription. You can find this in the Azure Dashboard under Home > Subscriptions. */
    private static String subscriptionId = "INSERT SUBSCRIPTION ID HERE";

    /* The Active Directory tenant ID. You can find this in the Azure Dashboard under Home > Azure Active Directory. */
    private static String tenantId = "INSERT TENANT ID HERE";

    /* The name of the Azure resource group in which you want to create the resource.
    You can find resource groups in the Azure Dashboard under Home > Resource groups. */
    private static String resourceGroupName = "INSERT RESOURCE GROUP NAME HERE";
    // </snippet_constants>

    public static void main(String[] args) {

        // <snippet_auth>
        // auth
        private static ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(applicationId, tenantId, applicationSecret, AzureEnvironment.AZURE);

        CognitiveServicesManager client = CognitiveServicesManager.authenticate(credentials, subscriptionId);
        // </snippet_auth>

        // <snippet_calls>
        // list all available resource kinds, SKUs, and locations for your Azure account.
        list_available_kinds_skus_locations (client);

        // list all resources for your Azure account.
        list_resources (client);

        // Create a resource with kind Text Translation, SKU F0 (free tier), location global.
        String resourceId = create_resource (client, "test_resource", resourceGroupName, "TextAnalytics", "S0", Region.US_WEST);

        // Delete the resource.
        delete_resource (client, resourceId);
    }
    // </snippet_calls>

    // <snippet_list_avail>
    public void list_available_kinds_skus_locations (CognitiveServicesManager client) {
        System.out.println ("Available SKUs:");
        System.out.println("Kind\tSKU Name\tSKU Tier\tLocations");
        ResourceSkus skus = client.resourceSkus();
        // See https://github.com/ReactiveX/RxJava/wiki/Blocking-Observable-Operators
        for (ResourceSku sku : skus.listAsync().toBlocking().toIterable()) {
            String locations = String.join (",", sku.locations());
            System.out.println (sku.kind() + "\t" + sku.name() + "\t" + sku.tier() + "\t" + locations);
        }
    }
    // </snippet_list_avail>

    // Note: Region values are listed in:
    // https://github.com/Azure/autorest-clientruntime-for-java/blob/master/azure-arm-client-runtime/src/main/java/com/microsoft/azure/arm/resources/Region.java
    // <snippet_create>
    public String create_resource (CognitiveServicesManager client, String resourceName, String resourceGroupName, String kind, String skuName, Region region) {
        System.out.println ("Creating resource: " + resourceName + "...");

        CognitiveServicesAccount result = client.accounts().define(resourceName)
            .withRegion(region)
            .withExistingResourceGroup(resourceGroupName)
            .withKind(kind)
            .withSku(new Sku().withName(skuName))
            .create();

        System.out.println ("Resource created.");
        System.out.println ("ID: " + result.id());
        System.out.println ("Provisioning state: " + result.properties().provisioningState().toString());
        System.out.println ();

        return result.id();
    }
    // </snippet_create>

    // <snippet_list>
    public void list_resources (CognitiveServicesManager client) {
        System.out.println ("Resources in resource group: " + resourceGroupName);
        // Note Azure resources are also sometimes referred to as accounts.
        Accounts accounts = client.accounts();
        for (CognitiveServicesAccount account : accounts.listByResourceGroupAsync(resourceGroupName).toBlocking().toIterable()) {
            System.out.println ("Kind: " + account.kind ());
            System.out.println ("SKU Name: " + account.sku().name());
            System.out.println ();
        }
    }
    // </snippet_list>
    
    // <snippet_delete>
    public void delete_resource (CognitiveServicesManager client, String resourceId) {
        System.out.println ("Deleting resource: " + resourceId + "...");
        client.accounts().deleteByIds (resourceId);
        System.out.println ("Resource deleted.");
        System.out.println ();
    }
    // </snippet_delete>
}
```

## <a name="see-also"></a>另请参阅

* [Azure 管理 SDK 参考文档](https://docs.microsoft.com/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable)
* [什么是 Azure 认知服务？](../../Welcome.md)
* [对 Azure 认知服务的请求进行身份验证](../../authentication.md)
* [使用 Azure 门户创建新资源](../../cognitive-services-apis-create-account.md)

