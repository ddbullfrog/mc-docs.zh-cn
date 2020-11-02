---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 08/26/2020
title: 在 Azure Databricks 上使用 Shiny - Azure Databricks
description: 了解如何使用 Shiny 包在 Azure Databricks 的 RStudio 内部开发交互式应用程序。
ms.openlocfilehash: d4749e93bfbabcfab46b8e9af6cfa0b0e7ebfd3e
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92473064"
---
# <a name="shiny-on-azure-databricks"></a>Azure Databricks 上的 Shiny

若构建交互式 R 应用程序和仪表板，可以使用 [Shiny](https://shiny.rstudio.com/)，这是 CRAN 上提供的一个开源 R 包，位于 Azure Databricks 群集上托管的 [RStudio Server](rstudio.md) 中。

有关 Shiny 用户指南中的许多交互式示例，请参阅 [Shiny 教程](https://shiny.rstudio.com/tutorial/)。

本文介绍如何在 Azure Databricks 上运行 Shiny 应用程序以及如何在 Shiny 应用程序中使用 Apache Spark。

## <a name="requirements"></a>要求

* （推荐）Databricks Runtime 6.2 或更高版本。 在 Databricks Runtime 6.1 及更低版本上，必须[安装 Shiny R 包](#frequently-asked-questions-faq)。
* [Azure Databricks 上的 RStudio](rstudio.md)。

## <a name="get-started-with-shiny"></a>开始使用 Shiny

1. 在 Azure Databricks 上打开 RStudio。
2. 在 RStudio 中，导入 Shiny 包并按以下方式运行示例应用 `01_hello`：

   ```r
   > library(shiny)
   > runExample("01_hello")

   Listening on http://127.0.0.1:3203
   ```

   将会弹出一个新窗口，显示 Shiny 应用程序。

   > [!div class="mx-imgBorder"]
   > ![第一个 Shiny 应用](../../../_static/images/shiny/shiny-01-hello.png)

## <a name="run-a-shiny-app-from-an-r-script"></a>通过 R 脚本运行 Shiny 应用

若要通过 R 脚本运行 Shiny 应用，请在 RStudio 编辑器中打开 R 脚本，然后点击右上角的“运行应用”按钮。

> [!div class="mx-imgBorder"]
> ![Shiny 运行应用](../../../_static/images/shiny/shiny-run-app.png)

## <a name="use-apache-spark-inside-shiny-apps"></a>在 Shiny 应用内部使用 Apache Spark

在 Azure Databricks 上开发 Shiny 应用程序时，可以使用 Apache Spark。 你可以使用 SparkR 和 sparklyr 与 Spark 进行交互。 你至少需要一名工作人员才能启动 Spark 任务。

以下示例使用 SparkR 启动 Spark 作业。 该示例使用 [ggplot2 钻石数据集](https://ggplot2.tidyverse.org/reference/diamonds.html)来按克拉绘制钻石价格。 可以使用应用程序顶部的滑块更改克拉范围，并且绘图的 x 轴范围也会相应更改。

```r
library(SparkR)
library(sparklyr)
library(dplyr)
library(ggplot2)
sparkR.session()

sc <- spark_connect(method = "databricks")
diamonds_tbl <- spark_read_csv(sc, path = "/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv")

# Define the UI
ui <- fluidPage(
  sliderInput("carat", "Select Carat Range:",
              min = 0, max = 5, value = c(0, 5), step = 0.01),
  plotOutput('plot')
)

# Define the server code
server <- function(input, output) {
  output$plot <- renderPlot({
    # Select diamonds in carat range
    df <- diamonds_tbl %>%
      dplyr::select("carat", "price") %>%
      dplyr::filter(carat >= !!input$carat[[1]], carat <= !!input$carat[[2]])

    # Scatter plot with smoothed means
    ggplot(df, aes(carat, price)) +
      geom_point(alpha = 1/2) +
      geom_smooth() +
      scale_size_area(max_size = 2) +
      ggtitle("Price vs. Carat")
  })
}

# Return a Shiny app object

shinyApp(ui = ui, server = server)
```

> [!div class="mx-imgBorder"]
> ![Spark Shiny 应用](../../../_static/images/shiny/shiny-spark.png)

## <a name="frequently-asked-questions-faq"></a>常见问题 (FAQ)

**可以在 Databricks Runtime 6.1 及更低版本上使用 Shiny 吗？**

是的。 若要在 Databricks Runtime 6.1 及更低版本上使用 Shiny，请将 Shiny 包作为 Azure Databricks [库](../../../libraries/workspace-libraries.md#cran-package)安装在群集上。 在 RStudio 控制台中使用 `install.packages(‘shiny’)` 或使用 RStudio 包管理器可能无法工作。

**为什么我的 Shiny 应用会在一段时间后“灰显”？**

如果没有与 Shiny 应用交互，则大约 4 分钟后将关闭与该应用的连接。

要重新连接，请刷新 Shiny 应用页面。 仪表板的状态将重置。

**为什么我的 Shiny 查看器窗口会在一段时间后消失？**

如果 Shiny 查看器窗口在空闲几分钟后消失，这是由于与“灰显”场景相同的超时造成的。

**我的应用启动后立即崩溃，但代码似乎是正确的。这是怎么回事？**

在 Azure Databricks 上的 Shiny 应用中可以显示的数据总量有 20 MB 的限制。 如果应用程序的总数据大小超过此限制，则启动后将立即崩溃。 为了避免这种情况，Databricks 建议减小数据大小，例如通过降低显示数据的采样率或降低图像的分辨率。

**为何长 Spark 作业从不会返回？**

这也是由于空闲超时。 运行时间超过前面提到的超时时间的任何 Spark 作业将无法呈现其结果，因为连接将在作业返回之前关闭。

**如何避免超时？**

* [此问题线程](https://github.com/rstudio/shiny/issues/2110#issuecomment-419971302)中建议了一种解决方法。 该解决方法在应用空闲时发送检测信号以使 WebSocket 保持活动状态。 但是，如果该应用被长时间运行的计算所阻止，则此解决方法将不起作用。
* Shiny 不支持长时间运行的任务。 Shiny 的博客文章建议使用 [promise 和 future](https://blog.rstudio.com/2018/06/26/shiny-1-1-0/) 来异步运行长任务，并保持不阻止应用。 这是一个使用检测信号使 Shiny 应用保持活动状态并在 `future` 构造中运行长时间运行的 Spark 作业的示例。

  ```r
  # Write an app that uses spark to access data on Databricks
  # First, install the following packages:
  install.packages(‘future’)
  install.packages(‘promises’)

  library(shiny)
  library(promises)
  library(future)
  plan(multisession)

  HEARTBEAT_INTERVAL_MILLIS = 1000  # 1 second

  # Define the long Spark job here
  run_spark <- function(x) {
    # Environment setting
    library("SparkR", lib.loc = "/databricks/spark/R/lib")
    sparkR.session()

    irisDF <- createDataFrame(iris)
    collect(irisDF)
    Sys.sleep(3)
    x + 1
  }

  run_spark_sparklyr <- function(x) {
    # Environment setting
    library(sparklyr)
    library(dplyr)
    library("SparkR", lib.loc = "/databricks/spark/R/lib")
    sparkR.session()
    sc <- spark_connect(method = "databricks")

    iris_tbl <- copy_to(sc, iris, overwrite = TRUE)
    collect(iris_tbl)
    x + 1
  }

  ui <- fluidPage(
    sidebarLayout(
      # Display heartbeat
      sidebarPanel(textOutput("keep_alive")),

      # Display the Input and Output of the Spark job
      mainPanel(
        numericInput('num', label = 'Input', value = 1),
        actionButton('submit', 'Submit'),
        textOutput('value')
      )
    )
  )
  server <- function(input, output) {
    #### Heartbeat ####
    # Define reactive variable
    cnt <- reactiveVal(0)
    # Define time dependent trigger
    autoInvalidate <- reactiveTimer(HEARTBEAT_INTERVAL_MILLIS)
    # Time dependent change of variable
    observeEvent(autoInvalidate(), {  cnt(cnt() + 1)  })
    # Render print
    output$keep_alive <- renderPrint(cnt())

    #### Spark job ####
    result <- reactiveVal() # the result of the spark job
    busy <- reactiveVal(0)  # whether the spark job is running
    # Launch a spark job in a future when actionButton is clicked
    observeEvent(input$submit, {
      if (busy() != 0) {
        showNotification("Already running Spark job...")
        return(NULL)
      }
      showNotification("Launching a new Spark job...")
      # input$num must be read outside the future
      input_x <- input$num
      fut <- future({ run_spark(input_x) }) %...>% result()
      # Or: fut <- future({ run_spark_sparklyr(input_x) }) %...>% result()
      busy(1)
      # Catch exceptions and notify the user
      fut <- catch(fut, function(e) {
        result(NULL)
        cat(e$message)
        showNotification(e$message)
      })
      fut <- finally(fut, function() { busy(0) })
      # Return something other than the promise so shiny remains responsive
      NULL
    })
    # When the spark job returns, render the value
    output$value <- renderPrint(result())
  }
  shinyApp(ui = ui, server = server)
  ```

**在开发过程中，一个 Shiny 应用链接可以接受多少个连接？**

Databricks 建议最多 20 个。

**可以使用与 Databricks Runtime 中安装的版本不同的 Shiny 包吗？**

是的。 请查阅[修正 R 包的版本](https://docs.microsoft.com/azure/databricks/kb/r/pin-r-packages)。

**如何开发可以发布到 Shiny 服务器的 Shiny 应用并访问 Azure Databricks 上的数据？**

虽然你可以在开发和测试 Azure Databricks期间使用 SparkR 或 sparklyr 自然地访问数据，但是在一个 Shiny 应用程序发布到独立托管服务之后，它不能直接访问 Azure Databricks 上的数据和表。

若要使你的应用程序在 Azure Databricks 外部运行，必须重写访问数据的方式。 下面是几个选项：

* 使用 [JDBC / ODBC](../../../integrations/bi/jdbc-odbc-bi.md) 将查询提交到 Azure Databricks 群集。
* 使用 [Databricks Connect](../../../dev-tools/databricks-connect.md)。
* 直接访问对象存储上的数据。

Databricks 建议你与 Azure Databricks 解决方案团队合作，为现有数据和分析体系结构找到最佳方法。

**如何保存在 Azure Databricks 上开发的 Shiny 应用程序？**

你可以通过 FUSE 挂载将应用程序代码保存[在 DBF 上](../../../data/databricks-file-system.md)，也可以将代码签入版本控制中。

**可以在 Azure Databricks 笔记本中开发 Shiny 应用程序吗？**

无法在 Azure Databricks 笔记本中开发 Shiny 应用程序。