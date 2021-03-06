## <a name="build-iot-edge"></a>生成 IoT Edge

本教程使用自定义 IoT Edge 模块与远程监视预配置解决方案通信。 因此，需从自定义源代码生成 IoT Edge 模块。 以下部分介绍如何安装 IoT Edge 并生成自定义 IoT Edge 模块。

### <a name="install-iot-edge"></a>安装 IoT Edge

以下步骤介绍如何在 Intel NUC 上安装预编译的 IoT Edge 软件：

1. 在 Intel NUC 上运行以下命令，以便配置所需的智能包存储库：

    ```bash
    smart channel --add IoT_Cloud type=rpm-md name="IoT_Cloud" baseurl=http://iotdk.intel.com/repos/iot-cloud/wrlinux7/rcpl13/ -y
    smart channel --add WR_Repo type=rpm-md baseurl=https://distro.windriver.com/release/idp-3-xt/public_feeds/WR-IDP-3-XT-Intel-Baytrail-public-repo/RCPL13/corei7_64/
    ```

    当命令提示“是否包括此通道?”时，输入 `y`。

1. 通过运行以下命令更新智能包管理器：

    ```bash
    smart update
    ```

1. 运行以下命令安装 Azure IoT Edge 包：

    ```bash
    smart config --set rpm-check-signatures=false
    smart install packagegroup-cloud-azure -y
    ```

1. 运行“Hello world”示例来验证安装。 该示例每隔五秒向 log.txt 文件写入 hello world 消息。 以下命令运行“Hello world”示例：

    ```bash
    cd /usr/share/azureiotgatewaysdk/samples/hello_world/
    ./hello_world hello_world.json
    ```

    停止示例时，忽略任何**无效自变量**消息。

    使用以下命令查看日志文件的内容：

    ```bash
    cat log.txt | more
    ```

### <a name="troubleshooting"></a>故障排除

如果收到错误“无任何包提供 util-linux-dev”，请尝试重新启动 Intel NUC。
