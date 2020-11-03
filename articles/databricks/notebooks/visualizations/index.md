---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 09/16/2020
title: 可视化效果 - Azure Databricks
description: 了解 Azure Databricks 笔记本支持的各种类型的可视化效果。
ms.openlocfilehash: 95f3fe809dd9bd101a650fb4a256fbe10b638d8c
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106704"
---
# <a name="visualizations"></a>可视化效果

Azure Databricks 使用 `display` 和 `displayHTML` 函数直接支持各种类型的可视化效果。

Azure Databricks 还以原生方式支持以 Python 和 R 编写的可视化库，并允许你安装和使用第三方库。

## <a name="display-function"></a>`display` 函数

`display` 函数支持多种数据和可视化类型。

### <a name="in-this-section"></a>本部分内容：

* [数据类型](#data-types)
  * [DataFrame](#dataframes)
  * [映像](#images)
  * [结构化流式处理数据帧](#structured-streaming-dataframes)
* [绘图类型](#plot-types)
  * [选择并配置图表类型](#choose-and-configure-a-chart-type)
  * [图表工具栏](#chart-toolbar)
  * [图表之间的颜色一致性](#color-consistency-across-charts)
  * [机器学习可视化效果](#machine-learning-visualizations)

### <a name="data-types"></a>数据类型

#### <a name="dataframes"></a>数据帧

在 Azure Databricks 中创建数据帧可视化效果的最简单方式是调用 `display(<dataframe-name>)`。 例如，如果你有一个钻石数据集的 Spark 数据帧 `diamonds_df`，该数据集已按钻石颜色进行分组。你要调用以下代码并计算平均价格

```python
from pyspark.sql.functions import avg
diamonds_df = spark.read.csv("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", header="true", inferSchema="true")

display(diamonds_df.select("color","price").groupBy("color").agg(avg("price")))
```

此时会显示一个表格，其中包含钻石颜色与平均价格。

> [!div class="mx-imgBorder"]
> ![钻石颜色与平均价格](../../_static/images/notebooks/diamonds-table.png)

> [!TIP]
>
> 如果在调用 `display` 函数后只看到 `OK` 而没有呈现可视化效果，原因很可能是传入的数据帧或集合为空。

`display()` 支持 [pandas 数据帧](https://pandas.pydata.org/pandas-docs/stable/getting_started/dsintro.html#dataframe). 如果引用 pandas 或 [koalas](../../languages/koalas.md) 数据帧但不指定 `display`，则会像在 Jupyter 中一样呈现表。

##### <a name="dataframe-display-method"></a><a id="dataframe-display-method"> </a><a id="df-display"> </a>数据帧 `display` 方法

> [!NOTE]
>
> 在 Databricks Runtime 7.1 及更高版本中可用。

[PySpark](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame)、[pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/overview.html) 和 [Koalas](../../languages/koalas.md) 数据帧有一个调用 Azure Databricks `display` 函数的 `display` 方法。 可以在执行简单的数据帧操作后调用该方法，例如：

```python
diamonds_df = spark.read.csv("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", header="true", inferSchema="true")
diamonds_df.select("color","price").display()
```

也可以在一系列链式数据帧操作结束时调用该方法，例如：

```python
from pyspark.sql.functions import avg
diamonds_df = spark.read.csv("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", header="true", inferSchema="true")

diamonds_df.select("color","price").groupBy("color").agg(avg("price")).display()
```

#### <a name="images"></a><a id="display-image-type"> </a><a id="images"> </a>图像

`display` 以富 HTML 的形式呈现包含图像数据类型的列。 `display` 尝试为匹配 Spark [ImageSchema](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.image.ImageSchema$@columnSchema:org.apache.spark.sql.types.StructType) 的 `DataFrame` 列呈现图像缩略图。
对于通过 [readImages](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.ml.image.ImageSchema$@readImages(path:String,sparkSession:org.apache.spark.sql.SparkSession,recursive:Boolean,numPartitions:Int,dropImageFailures:Boolean,sampleRatio:Double,seed:Long):org.apache.spark.sql.DataFrame) 函数成功读入的任何图像，可以正常运行缩略图呈现。 对于通过其他方式生成的图像值，Azure Databricks 支持呈现 1、3 或 4 通道图像（每个通道由一个字节组成），但存在以下约束：

* **单通道图像** ：`mode` 字段必须等于 0。 `height`、`width` 和 `nChannels` 字段必须准确描述 `data` 字段中的二进制图像数据
* **三通道图像** ：`mode` 字段必须等于 16。 `height`、`width` 和 `nChannels` 字段必须准确描述 `data` 字段中的二进制图像数据。 `data` 字段必须包含三字节区块形式的像素数据，每个像素的通道顺序为 `(blue, green, red)`。
* **四通道图像** ：`mode` 字段必须等于 24。 `height`、`width` 和 `nChannels` 字段必须准确描述 `data` 字段中的二进制图像数据。 `data` 字段必须包含四字节区块形式的像素数据，每个像素的通道顺序为 `(blue, green, red, alpha)`。

##### <a name="example"></a>示例

假设某个文件夹包含一些图像：

> [!div class="mx-imgBorder"]
> ![图像数据的文件夹](../../_static/images/notebooks/sample-image-data.png)

如果使用 `ImageSchema.readImages` 将图像读入数据帧，然后显示数据帧，则 `display` 会呈现图像的缩略图：

```python
from pyspark.ml.image import ImageSchema
image_df = ImageSchema.readImages(sample_img_dir)
display(image_df)
```

> [!div class="mx-imgBorder"]
> ![显示图像数据帧](../../_static/images/notebooks/image-data.png)

#### <a name="structured-streaming-dataframes"></a><a id="ss-display"> </a><a id="structured-streaming-dataframes"> </a>结构化流式处理数据帧

若要实时地直观显示流式处理查询的结果，可以将 Scala 和 Python 中的 `display` 用于结构化流式处理数据帧。

##### <a name="python"></a>Python

```python
streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count())
```

##### <a name="scala"></a>Scala

```scala
val streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count())
```

`display` 支持以下可选参数：

* `streamName`：流式处理查询名称。
* `trigger` (Scala) 和 `processingTime` (Python)：定义运行流式处理查询的频率。 如果未指定，则系统会在上一处理完成后立即检查是否有新数据可用。 为了降低生产成本，我们建议你总是设置一个触发器时间间隔。
* `checkpointLocation`：系统写入所有检查点信息的位置。 如果未指定，则系统会在 DBFS 上自动生成一个临时检查点位置。 为了使你的流可以继续从中断的位置处理数据，你必须提供一个检查点位置。 建议在生产环境中总是指定 `checkpointLocation` 选项。

##### <a name="python"></a>Python

```python
streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count(), processingTime = "5 seconds", checkpointLocation = "dbfs:/<checkpoint-path>")
```

##### <a name="scala"></a>Scala

```scala
import org.apache.spark.sql.streaming.Trigger

val streaming_df = spark.readStream.format("rate").load()
display(streaming_df.groupBy().count(), trigger = Trigger.ProcessingTime("5 seconds"), checkpointLocation = "dbfs:/<checkpoint-path>")
```

有关这些参数的详细信息，请参阅[启动流式处理查询](http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#starting-streaming-queries)。

### <a name="plot-types"></a>绘图类型

`display` 函数支持一组丰富的绘图类型：

> [!div class="mx-imgBorder"]
> ![图表类型](../../_static/images/notebooks/display-charts.png)

#### <a name="choose-and-configure-a-chart-type"></a>选择并配置图表类型

若要选择条形图，请单击“条形图”图标 ![图表按钮](../../_static/images/notebooks/chart-button.png):

> [!div class="mx-imgBorder"]
> ![条形图图标](../../_static/images/notebooks/diamonds-bar-chart.png)

若要选择其他绘图类型，请单击 ![下拉按钮](../../_static/images/button-down.png) （位于条形图 ![图表按钮](../../_static/images/notebooks/chart-button.png) 右侧），然后选择绘图类型。

#### <a name="chart-toolbar"></a>图表工具栏

折线图和条形图都具有内置工具栏，该工具栏支持一组丰富的客户端交互。

> [!div class="mx-imgBorder"]
> ![图表工具栏](../../_static/images/notebooks/chart-toolbar.png)

若要配置图表，请单击“绘图选项…”。

> [!div class="mx-imgBorder"]
> ![绘图选项](../../_static/images/notebooks/plot-options.png)

折线图具有多个自定义图表选项：设置 Y 轴范围、显示和隐藏点，以及显示带有对数刻度的 Y 轴。

有关旧图表类型的信息，请参阅：

* [旧版折线图](legacy-charts.md)

#### <a name="color-consistency-across-charts"></a>图表之间的颜色一致性

Azure Databricks 支持图表中的两种颜色一致性：系列集和全局。

如果系列的值相同但顺序不同（例如，A = `["Apple", "Orange", "Banana"]`，B = `["Orange", "Banana", "Apple"]`），则“系列集”颜色一致性会将相同的颜色分配给相同的值。 这些值在绘制之前已排序，因此，两个图例的排序方式相同 (`["Apple", "Banana", "Orange"]`)，并为相同的值分配相同的颜色。 但是，如果系列为 C = `["Orange", "Banana"]`，则它的颜色与集 A 不一致，因为该集是不相同的。 排序算法会将第一种颜色分配给集 C 中的“Banana”，将第二种颜色分配给集 A 中的“Banana”。如果希望这些系列的颜色一致，可以指定图表使用全局颜色一致性。

在“全局”颜色一致性中，无论系列的值是什么，每个值都始终映射到相同的颜色。 若要为每个图表启用此行为，请选中“全局颜色一致性”复选框。

> [!div class="mx-imgBorder"]
> ![全局颜色一致性](../../_static/images/notebooks/series-colors.gif)

> [!NOTE]
>
> 为了实现这种一致性，Azure Databricks 会直接从值哈希处理到颜色。 为了避免冲突（两个值的颜色完全相同），哈希处理将对较大的颜色集进行，但这会造成这样的负面影响：无法保证颜色的鲜艳或易于分辨性；如果颜色过多，在一定程度上它们看上去会很相似。

#### <a name="machine-learning-visualizations"></a><a id="display-ml-models"> </a><a id="machine-learning-visualizations"> </a>机器学习可视化效果

除了标准图表类型外，`display` 函数还支持以下机器学习训练参数和结果的可视化：

* [残差](#residuals)
* [ROC 曲线](#roc-curves)
* [决策树](#decision-trees)

##### <a name="residuals"></a>残差

对于线性回归和逻辑回归，`display` 支持呈现[拟合与残差](https://en.wikipedia.org/wiki/Errors_and_residuals)绘图。 若要获取此绘图，请提供模型和数据帧。

以下示例对城市人口进行线性回归以容纳售价数据，然后显示残差与拟合数据。

```python
# Load data
pop_df = spark.read.csv("/databricks-datasets/samples/population-vs-price/data_geo.csv", header="true", inferSchema="true")

# Drop rows with missing values and rename the feature and label columns, replacing spaces with _
pop_df = data.dropna() # drop rows with missing values
exprs = [col(column).alias(column.replace(' ', '_')) for column in data.columns]

# Register a UDF to convert the feature (2014_Population_estimate) column vector to a VectorUDT type and apply it to the column.
from pyspark.ml.linalg import Vectors, VectorUDT

spark.udf.register("oneElementVec", lambda d: Vectors.dense([d]), returnType=VectorUDT())
tdata = data.select(*exprs).selectExpr("oneElementVec(2014_Population_estimate) as features", "2015_median_sales_price as label")

# Run a linear regression
from pyspark.ml.regression import LinearRegression

lr = LinearRegression()
modelA = lr.fit(tdata, {lr.regParam:0.0})

# Plot residuals versus fitted data
display(modelA, tdata)
```

> [!div class="mx-imgBorder"]
> ![显示残差](../../_static/images/notebooks/residuals.png)

##### <a name="roc-curves"></a>ROC 曲线

对于逻辑回归，`display` 支持呈现 [ROC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) 曲线。 若要获取此绘图，请提供模型、用作 `fit` 方法的输入的已准备数据，以及参数 `"ROC"`。

以下示例开发一个分类器，用于基于个人的各种属性预测此人在一年中的收入是 <=50K 还是 >50K。 成年人数据集派生自人口统计数据，包括有关 48842 个人及其每年收入的信息。

```sql
CREATE TABLE adult (
  age DOUBLE,
  workclass STRING,
  fnlwgt DOUBLE,
  education STRING,
  education_num DOUBLE,
  marital_status STRING,
  occupation STRING,
  relationship STRING,
  race STRING,
  sex STRING,
  capital_gain DOUBLE,
  capital_loss DOUBLE,
  hours_per_week DOUBLE,
  native_country STRING,
  income STRING)
USING CSV
OPTIONS (path "/databricks-datasets/adult/adult.data", header "true")
```

```python
dataset = spark.table("adult")

# Use One-Hot Encoding to convert all categorical variables into binary vectors.

from pyspark.ml import Pipeline
from pyspark.ml.feature import OneHotEncoderEstimator, StringIndexer, VectorAssembler
categoricalColumns = ["workclass", "education", "marital_status", "occupation", "relationship", "race", "sex", "native_country"]

stages = [] # stages in our Pipeline
for categoricalCol in categoricalColumns:
    # Category Indexing with StringIndexer
    stringIndexer = StringIndexer(inputCol=categoricalCol, outputCol=categoricalCol + "Index")
    # Use OneHotEncoder to convert categorical variables into binary SparseVectors
    # encoder = OneHotEncoderEstimator(inputCol=categoricalCol + "Index", outputCol=categoricalCol + "classVec")
    encoder = OneHotEncoderEstimator(inputCols=[stringIndexer.getOutputCol()], outputCols=[categoricalCol + "classVec"])
    # Add stages.  These are not run here, but will run all at once later on.
    stages += [stringIndexer, encoder]

# Convert label into label indices using the StringIndexer
label_stringIdx = StringIndexer(inputCol="income", outputCol="label")
stages += [label_stringIdx]

# Transform all features into a vector using VectorAssembler
numericCols = ["age", "fnlwgt", "education_num", "capital_gain", "capital_loss", "hours_per_week"]
assemblerInputs = [c + "classVec" for c in categoricalColumns] + numericCols
assembler = VectorAssembler(inputCols=assemblerInputs, outputCol="features")
stages += [assembler]

# Run the stages as a Pipeline. This puts the data through all of the feature transformations in a single call.

partialPipeline = Pipeline().setStages(stages)
pipelineModel = partialPipeline.fit(dataset)
preppedDataDF = pipelineModel.transform(dataset)

# Fit logistic regression model

from pyspark.ml.classification import LogisticRegression
lrModel = LogisticRegression().fit(preppedDataDF)

# ROC for data
display(lrModel, preppedDataDF, "ROC")
```

> [!div class="mx-imgBorder"]
> ![显示 ROC](../../_static/images/notebooks/roc.png)

若要显示残差，请省略 `"ROC"` 参数：

```python
display(lrModel, preppedDataDF)
```

> [!div class="mx-imgBorder"]
> ![显示残差](../../_static/images/notebooks/log-reg-residuals.png)

##### <a name="decision-trees"></a>决策树

`display` 函数支持呈现决策树。

若要获取此可视化效果，请提供决策树模型。

以下示例对某个树进行训练，以从手写数字图像的 MNIST 数据集中识别数字 (0 - 9)，然后显示该树。

###### <a name="python"></a>Python

```python
trainingDF = spark.read.format("libsvm").load("/databricks-datasets/mnist-digits/data-001/mnist-digits-train.txt").cache()
testDF = spark.read.format("libsvm").load("/databricks-datasets/mnist-digits/data-001/mnist-digits-test.txt").cache()

from pyspark.ml.classification import DecisionTreeClassifier
from pyspark.ml.feature import StringIndexer
from pyspark.ml import Pipeline

indexer = StringIndexer().setInputCol("label").setOutputCol("indexedLabel")

dtc = DecisionTreeClassifier().setLabelCol("indexedLabel")

# Chain indexer + dtc together into a single ML Pipeline.
pipeline = Pipeline().setStages([indexer, dtc])

model = pipeline.fit(trainingDF)
display(model.stages[-1])
```

###### <a name="scala"></a>Scala

```scala
val trainingDF = spark.read.format("libsvm").load("/databricks-datasets/mnist-digits/data-001/mnist-digits-train.txt").cache
val testDF = spark.read.format("libsvm").load("/databricks-datasets/mnist-digits/data-001/mnist-digits-test.txt").cache

import org.apache.spark.ml.classification.{DecisionTreeClassifier, DecisionTreeClassificationModel}
import org.apache.spark.ml.feature.StringIndexer
import org.apache.spark.ml.Pipeline

val indexer = new StringIndexer().setInputCol("label").setOutputCol("indexedLabel")
val dtc = new DecisionTreeClassifier().setLabelCol("indexedLabel")
val pipeline = new Pipeline().setStages(Array(indexer, dtc))

val model = pipeline.fit(trainingDF)
val tree = model.stages.last.asInstanceOf[DecisionTreeClassificationModel]

display(tree)
```

> [!div class="mx-imgBorder"]
> ![显示决策树](../../_static/images/notebooks/decision-tree.png)

## <a name="displayhtml-function"></a><a id="display-html-function"> </a><a id="displayhtml-function"> </a>`displayHTML` 函数

Azure Databricks 编程语言笔记本（Python、R 和 Scala）使用 `displayHTML` 函数支持 HTML 图形；你可以传递任何 HTML、CSS 或 JavaScript 代码。 此函数使用 JavaScript 库（例如 D3）支持交互式图形。

有关使用 `displayHTML` 的示例，请参阅：

* [笔记本中的 HTML、D3 和 SVG](html-d3-and-svg.md)

* [在笔记本中嵌入静态图像](../../data/filestore.md#static-images)

> [!NOTE]
>
> `displayHTML` iframe 是从域 `databricksusercontent.com` 提供的，iframe 沙盒包含 `allow-same-origin` 属性。 必须可在浏览器中访问 `databricksusercontent.com`。 如果它当前被企业网络阻止，IT 人员需要将它加入允许列表。

## <a name="visualizations-by-language"></a>可视化（按语言）

### <a name="in-this-section"></a>本部分内容：

* [Python 中的可视化效果](#visualizations-in-python)
* [使用 R 进行可视化](#visualizations-in-r)
* [Scala 中的可视化效果](#visualizations-in-scala)
* [使用 SQL 进行可视化](#visualizations-in-sql)

### <a name="visualizations-in-python"></a>Python 中的可视化效果

若要通过 Python 为数据绘图，请使用 `display` 函数，如下所示：

```python
diamonds_df = spark.read.csv("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", header="true", inferSchema="true")

display(diamonds_df.groupBy("color").avg("price").orderBy("color"))
```

> [!div class="mx-imgBorder"]
> ![Python 条形图](../../_static/images/notebooks/diamonds-bar-chart.png)

#### <a name="in-this-section"></a>本部分内容：

* [深入了解 Python 笔记本](#deep-dive-python-notebook)
* [Seaborn](#seaborn)
* [其他 Python 库](#other-python-libraries)

#### <a name="deep-dive-python-notebook"></a>深入了解 Python 笔记本

若要深入了解如何使用 `display` 实现 Python 可视化效果，请参阅笔记本：

* [深入探讨使用 Python 进行可视化](charts-and-graphs-python.md)

#### <a name="seaborn"></a>Seaborn

还可以使用其他 Python 库来生成绘图。 Databricks Runtime 包含 [seaborn](https://seaborn.pydata.org/) 可视化库。 若要创建 seaborn 绘图，请导入库，创建绘图，然后将该绘图传递给 `display` 函数。

```python
import seaborn as sns
sns.set(style="white")

df = sns.load_dataset("iris")
g = sns.PairGrid(df, diag_sharey=False)
g.map_lower(sns.kdeplot)
g.map_diag(sns.kdeplot, lw=3)

g.map_upper(sns.regplot)

display(g.fig)
```

> [!div class="mx-imgBorder"]
> ![Seaborn 绘图](../../_static/images/notebooks/seaborn-iris.png)

#### <a name="other-python-libraries"></a>其他 Python 库

* [Bokeh](bokeh.md)
* [Matplotlib](matplotlib.md)
* [Plotly](plotly.md)

### <a name="visualizations-in-r"></a>R 中的可视化效果

若要通过 R 为数据绘图，请使用 `display` 函数，如下所示：

```r
library(SparkR)
diamonds_df <- read.df("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv", source = "csv", header="true", inferSchema = "true")

display(arrange(agg(groupBy(diamonds_df, "color"), "price" = "avg"), "color"))
```

可以使用默认的 R [plot](https://www.rdocumentation.org/packages/graphics/versions/3.6.2/topics/plot) 函数。

```r
fit <- lm(Petal.Length ~., data = iris)
layout(matrix(c(1,2,3,4),2,2)) # optional 4 graphs/page
plot(fit)
```

> [!div class="mx-imgBorder"]
> ![默认的 R plot 绘图](../../_static/images/notebooks/r-iris.png)

还可以使用任何 R 可视化包。 R 笔记本以 `.png` 形式捕获生成的绘图，并以内联方式显示它。

#### <a name="in-this-section"></a>本部分内容：

* [Lattice](#lattice)
* [DandEFA](#dandefa)
* [Plotly](#plotly)
* [其他 R 库](#other-r-libraries)

#### <a name="lattice"></a>Lattice

[Lattice](https://www.statmethods.net/advgraphs/trellis.html) 包支持网格图，这些图用于显示变量或变量之间的关系（该关系取决于一个或多个其他变量）。

```r
library(lattice)
xyplot(price ~ carat | cut, diamonds, scales = list(log = TRUE), type = c("p", "g", "smooth"), ylab = "Log price")
```

> [!div class="mx-imgBorder"]
> ![R Lattice 绘图](../../_static/images/notebooks/r-lattice.png)

#### <a name="dandefa"></a>DandEFA

[DandEFA](https://www.rdocumentation.org/packages/DandEFA/versions/1.6) 包支持 dandelion 绘图。

```r
install.packages("DandEFA", repos = "https://cran.us.r-project.org")
library(DandEFA)
data(timss2011)
timss2011 <- na.omit(timss2011)
dandpal <- rev(rainbow(100, start = 0, end = 0.2))
facl <- factload(timss2011,nfac=5,method="prax",cormeth="spearman")
dandelion(facl,bound=0,mcex=c(1,1.2),palet=dandpal)
facl <- factload(timss2011,nfac=8,method="mle",cormeth="pearson")
dandelion(facl,bound=0,mcex=c(1,1.2),palet=dandpal)
```

> [!div class="mx-imgBorder"]
> ![R DandEFA 绘图](../../_static/images/notebooks/r-daefa.png)

#### <a name="plotly"></a>Plotly

[Plotly](https://plotly.com/r/) R 包依赖于 [htmlwidgets for R](https://www.htmlwidgets.org/)。有关安装说明和笔记本，请参阅 [htmlwidgets](htmlwidgets.md)。

#### <a name="other-r-libraries"></a>其他 R 库

* [ggplot2](ggplot2.md)
* [htmlwidgets](htmlwidgets.md)

### <a name="visualizations-in-scala"></a>Scala 中的可视化效果

若要通过 Scala 为数据绘图，请使用 `display` 函数，如下所示：

```scala
val diamonds_df = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv")

display(diamonds_df.groupBy("color").avg("price").orderBy("color"))
```

> [!div class="mx-imgBorder"]
> ![Scala 条形图](../../_static/images/notebooks/diamonds-bar-chart.png)

#### <a name="deep-dive-scala-notebook"></a>深入了解 Scala 笔记本

若要深入了解如何使用 `display` 实现 Python 可视化效果，请参阅笔记本：

* [深入探讨使用 Scala 进行可视化](charts-and-graphs-scala.md)

### <a name="visualizations-in-sql"></a><a id="sql-viz"> </a><a id="visualizations-in-sql"> </a>SQL 中的可视化效果

运行 SQL 查询时，Azure Databricks 将自动提取某些数据并以表格形式显示这些数据。

```sql
SELECT color, avg(price) AS price FROM diamonds GROUP BY color ORDER BY COLOR
```

> [!div class="mx-imgBorder"]
> ![SQL 表](../../_static/images/notebooks/diamonds-table.png)

从中可以选择不同的图表类型。

> [!div class="mx-imgBorder"]
> ![SQL 条形图](../../_static/images/notebooks/diamonds-bar-chart.png)