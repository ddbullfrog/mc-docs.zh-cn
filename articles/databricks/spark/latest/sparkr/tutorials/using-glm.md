---
ms.topic: conceptual
ms.service: azure-databricks
ms.reviewer: mamccrea
ms.custom: databricksmigration
ms.author: saperla
author: mssaperla
ms.date: 03/17/2020
title: 使用 glm - Azure Databricks
description: 了解如何在 Azure Databricks 中使用通用化线性模型 (GLM) 执行线性和逻辑回归。
ms.openlocfilehash: 1b2492ba701b29e9ddc575fb662ddcd9e5fd69f9
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472773"
---
# <a name="use-glm"></a>使用 glm

`glm` 拟合通用线性模型，这是类似于 R 的 glm()。

**语法** ：`glm(formula, data, family...)`

**参数** ：

* `formula`：要拟合的模型的符号说明，例如：`ResponseVariable ~ Predictor1 + Predictor2`。 支持的运算符：`~`、`+`、`-` 和 `.`
* `data`：任何 SparkDataFrame
* `family`：字符串 `"gaussian"`（用于线性回归）或 `"binomial"`（用于逻辑回归）
* `lambda`：数值，正则化参数
* `alpha`：数值，弹性网络混合参数

**输出** ：MLlib PipelineModel

本教程介绍如何对钻石数据集执行线性回归和逻辑回归。

## <a name="load-diamonds-data-and-split-into-training-and-test-sets"></a>加载钻石数据并将其拆分为训练集和测试集

```r
require(SparkR)

# Read diamonds.csv dataset as SparkDataFrame
diamonds <- read.df("/databricks-datasets/Rdatasets/data-001/csv/ggplot2/diamonds.csv",
                  source = "com.databricks.spark.csv", header="true", inferSchema = "true")
diamonds <- withColumnRenamed(diamonds, "", "rowID")

# Split data into Training set and Test set
trainingData <- sample(diamonds, FALSE, 0.7)
testData <- except(diamonds, trainingData)

# Exclude rowIDs
trainingData <- trainingData[, -1]
testData <- testData[, -1]

print(count(diamonds))
print(count(trainingData))
print(count(testData))
```

```r
head(trainingData)
```

## <a name="train-a-linear-regression-model-using-glm"></a>使用 `glm()` 训练线性回归模型

本部分介绍了如何通过使用训练数据训练线性回归模型来根据钻石的特征预测钻石的价格。

将分类特征（切割 - 理想、高级、非常好...）和连续特征（深度，克拉）混合在一起。
在内部，SparkR 会自动对此类特征执行独热编码，这样便不必手动执行此操作。

```r
# Family = "gaussian" to train a linear regression model
lrModel <- glm(price ~ ., data = trainingData, family = "gaussian")

# Print a summary of the trained model
summary(lrModel)
```

对测试数据使用 `predict()`，以查看模型对新数据的处理情况。

**语法** ：`predict(model, newData)`

**参数** ：

* `model`：MLlib 模型
* `newData`：SparkDataFrame，通常为你的测试集

**输出** ：SparkDataFrame

```r
# Generate predictions using the trained model
predictions <- predict(lrModel, newData = testData)

# View predictions against mpg column
display(select(predictions, "price", "prediction"))
```

评估模型。

```r
errors <- select(predictions, predictions$price, predictions$prediction, alias(predictions$price - predictions$prediction, "error"))
display(errors)

# Calculate RMSE
head(select(errors, alias(sqrt(sum(errors$error^2 , na.rm = TRUE) / nrow(errors)), "RMSE")))
```

## <a name="train-a-logistic-regression-model-using-glm"></a>使用 `glm()` 训练逻辑回归模型

本部分说明如何在同一数据集上创建逻辑回归，以根据钻石的某些特征预测钻石的切割。

MLlib 中的逻辑回归只支持二元分类。 若要在此示例中测试算法，请对数据进行子集处理以仅使用 2 个标签。

```r
# Subset data to include rows where diamond cut = "Premium" or diamond cut = "Very Good"
trainingDataSub <- subset(trainingData, trainingData$cut %in% c("Premium", "Very Good"))
testDataSub <- subset(testData, testData$cut %in% c("Premium", "Very Good"))
```

```r
# Family = "binomial" to train a logistic regression model
logrModel <- glm(cut ~ price + color + clarity + depth, data = trainingDataSub, family = "binomial")

# Print summary of the trained model
summary(logrModel)
```

```r
# Generate predictions using the trained model
predictionsLogR <- predict(logrModel, newData = testDataSub)

# View predictions against label column
display(select(predictionsLogR, "label", "prediction"))
```

评估模型。

```r
errorsLogR <- select(predictionsLogR, predictionsLogR$label, predictionsLogR$prediction, alias(abs(predictionsLogR$label - predictionsLogR$prediction), "error"))
display(errorsLogR)
```