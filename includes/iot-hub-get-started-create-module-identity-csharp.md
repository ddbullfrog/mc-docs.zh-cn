---
title: include 文件
description: include 文件
services: iot-hub
author: chrissie926
manager: timlt
ms.service: iot-hub
ms.topic: include
ms.date: 04/26/2018
ms.author: menchi
ms.custom: include file
ms.openlocfilehash: f3551cdfb4a7abccbfd6a39fce5d8474b99221ff
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "69993084"
---
## <a name="create-a-module-identity"></a>创建模块标识

本部分将创建一个 .NET 控制台应用，用于在中心的标识注册表中创建设备标识和模块标识。 设备或模块无法连接到中心，除非它在标识注册表中具有条目。 有关详细信息，请参阅 [IoT 中心开发人员指南的“标识注册表”部分](../articles/iot-hub/iot-hub-devguide-identity-registry.md)。

运行此控制台应用时，它会为设备和模块生成唯一的 ID 和密钥。 设备和模块在向 IoT 中心发送设备到云的消息时，使用这些值来标识自身。 ID 区分大小写。

1. 打开 Visual Studio，选择“新建项目”  。

1. 在“创建新项目”中，选择“控制台应用(.NET Framework)”   。

1. 选择“下一步”，  打开“配置新项目”。  将项目命名为 CreateIdentities，将解决方案命名为 IoTHubGetStarted   。 确保 .NET Framework 版本为 4.6.1 或更高。

    ![输入 Visual Studio 解决方案的名称和框架](./media/iot-hub-get-started-create-module-identity-csharp/configure-createidentities-project.png)

1. 在 Visual Studio 中，打开“工具” > “NuGet 包管理器” > “管理解决方案的 NuGet 包”。 选择“浏览”按钮  。

1. 搜索 **Microsoft.Azure.Devices**。 选择它，然后选择“安装”。 

    ![安装 Azure IoT 中心 .NET 服务 SDK 当前版本](./media/iot-hub-get-started-create-module-identity-csharp/install-service-sdk.png)

1. 在 Program.cs 文件顶部添加以下 `using` 语句：

    ```csharp
    using Microsoft.Azure.Devices;
    using Microsoft.Azure.Devices.Common.Exceptions;
    ```

4. 将以下字段添加到 Program 类  。 将占位符值替换为在上一部分为中心创建的 IoT 中心连接字符串。

    ```csharp
    const string connectionString = "<replace_with_iothub_connection_string>";
    const string deviceID = "myFirstDevice";
    const string moduleID = "myFirstModule";
    ```

5. 将以下代码添加到 Main  类。
    ```csharp
    static void Main(string[] args)
    {
        AddDeviceAsync().Wait();
        AddModuleAsync().Wait();
    }
    ```

6. 将以下方法添加到 Program  类：

    ```csharp
    private static async Task AddDeviceAsync()
    {
        RegistryManager registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        Device device;

        try
        {
            device = await registryManager.AddDeviceAsync(new Device(deviceID));
        }
        catch (DeviceAlreadyExistsException)
        {
            device = await registryManager.GetDeviceAsync(deviceID);
            }

            Console.WriteLine("Generated device key: {0}", device.Authentication.SymmetricKey.PrimaryKey);
    }

    private static async Task AddModuleAsync()
    {
        RegistryManager registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        Module module;

        try
        {
            module = await registryManager.AddModuleAsync(new Module(deviceID, moduleID));
        }
        catch (ModuleAlreadyExistsException)
        {
            module = await registryManager.GetModuleAsync(deviceID, moduleID);
        }

        Console.WriteLine("Generated module key: {0}", module.Authentication.SymmetricKey.PrimaryKey);
    }
    ```

    `AddDeviceAsync` 方法会创建 ID 为 **myFirstDevice** 的设备标识。 如果该设备 ID 已在标识注册表中，代码就只检索现有的设备信息。 然后，应用程序会显示该标识的主密钥。 在模拟设备应用中使用此密钥连接到中心。

    `AddModuleAsync` 方法在设备 **myFirstDevice** 下创建 ID 为 **myFirstModule** 的模块标识。 如果该模块 ID 已在标识注册表中，代码就只检索现有的模块信息。 然后，应用程序会显示该标识的主密钥。 在模拟模块应用中使用此密钥连接到中心。

[!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

6. 运行此应用程序并记下设备密钥和模块密钥。

> [!NOTE]
> IoT 中心标识注册表只存储设备和模块标识，以启用对 IoT 中心的安全访问。 标识注册表存储用作安全凭据的设备 ID 和密钥。 标识注册表还为每个设备存储启用/禁用标志，该标志可以用于禁用对该设备的访问。 如果应用程序需要存储其他特定于设备的元数据，则应使用特定于应用程序的存储。 没有针对模块标识的“已启用/已禁用”标记。 有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

<!-- Images. -->
[10]: ./media/iot-hub-get-started-create-module-identity-csharp/install-sdk.png
[11]: ./media/iot-hub-get-started-create-module-identity-csharp/create-identities-csharp1.JPG

<!-- Links -->
[lnk-devguide-identity]: ../articles/iot-hub/iot-hub-devguide-identity-registry.md
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
