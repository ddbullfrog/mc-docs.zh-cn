---
title: 评估 AutoML 试验结果
titleSuffix: Azure Machine Learning
description: 了解如何查看和评估每个自动化机器学习试验运行的图表与指标。
services: machine-learning
author: aniththa
ms.author: anumamah
ms.reviewer: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.date: 10/09/2020
ms.topic: conceptual
ms.custom: how-to, contperfq2
ms.openlocfilehash: 3f77dc8a44dee2bd5835d3ee477e0dc0fe7ec8fc
ms.sourcegitcommit: 93309cd649b17b3312b3b52cd9ad1de6f3542beb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93106070"
---
# <a name="evaluate-automated-machine-learning-experiment-results"></a>评估自动化机器学习试验结果

本文介绍如何查看和评估自动化机器学习、AutoML 和试验的结果。 这些试验包含多个运行，其中每个运行都会创建一个模型。 为了帮助你评估每个模型，AutoML 会自动生成特定于试验类型的性能指标和图表。 

例如，AutoML 为分类和回归模型提供不同的图表。 

|分类|回归
|---|---|
|<li> [混淆矩阵](#confusion-matrix) <li>[精度-召回率图表](#precision-recall-chart) <li> [接收方操作特征 (ROC)](#roc) <li> [提升曲线](#lift-curve)<li> [增益曲线](#gains-curve)<li> [校准图](#calibration-plot) | <li> [预测与真实](#pvt) <li> [残差直方图](#histo)|

## <a name="prerequisites"></a>先决条件

* Azure 订阅。 如果没有 Azure 订阅，请在开始前创建一个试用帐户。 立即试用[免费版或付费版 Azure 机器学习](https://www.azure.cn/pricing/1rmb-trial)。

* 使用 SDK 或 Azure 机器学习工作室为自动化机器学习运行创建试验。

    * 使用 SDK 生成[分类模型](how-to-auto-train-remote.md)或[回归模型](tutorial-auto-train-models.md)
    * 使用 [Azure 机器学习工作室](how-to-use-automated-ml-for-ml-models.md)通过上传相应的数据来创建分类模型或回归模型。

## <a name="view-run-results"></a>查看运行结果

自动化机器学习试验完成后，可通过 [Azure 机器学习工作室](overview-what-is-machine-learning-studio.md)，在机器学习工作区中找到运行历史记录。 

对于 SDK 试验，在使用 `RunDetails` [Jupyter 小组件](https://docs.microsoft.com/python/api/azureml-widgets/azureml.widgets?view=azure-ml-py&preserve-view=true)时，可在运行期间看到相同的结果。

以下步骤和动画演示了如何在工作室中查看运行历史记录以及特定模型的性能指标和图表。

![查看运行历史记录、模型性能指标和图表的步骤](./media/how-to-understand-automated-ml/view-run-metrics-ui.gif)

若要在工作室中查看运行历史记录、模型性能指标和图表： 

1. [登录到工作室](https://ml.azure.com/)并导航到你的工作区。
1. 在工作区的左侧面板中，选择“运行”。
1. 在试验列表中，选择要探索的试验。
1. 在底部的表中，选择“运行”。
1. 在“模型”选项卡中，选择要浏览的模型的“算法名称” 。
1. 在“指标”选项卡上，选择要为该模型评估的指标和图表。 


<a name="classification"></a> 

## <a name="classification-performance-metrics"></a>分类性能指标

下表总结了 AutoML 针对为试验生成的每个分类模型计算的模型性能指标。 

指标|说明|计算|其他参数
--|--|--|--
AUC_macro| AUC 是接收方操作特性曲线下面的区域。 Macro 是每个类的 AUC 算术平均值。  | [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="macro"|
AUC_micro| AUC 是接收方操作特性曲线下面的区域。 通过组合每个类中的真报率和误报率来全局计算 Micro。| [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="micro"|
AUC_weighted  | AUC 是接收方操作特性曲线下面的区域。 Weighted 是每个类的评分算术平均值，按每个类中的真实实例数加权。| [计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html)|average="weighted"
accuracy|Accuracy 是与真实标签完全匹配的预测标签百分比。 |[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html) |无|
average_precision_score_macro|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 Macro 是每个类的平均精度评分算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="macro"|
average_precision_score_micro|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 通过组合每个交接中的真报率和误报率来全局计算 Micro。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="micro"|
average_precision_score_weighted|平均精度以每个阈值实现的加权精度汇总精度-召回率曲线，使用前一阈值中的召回率增量作为权重。 Weighted 是每个类的平均精度评分算术平均值，按每个类中的真实实例数加权。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="weighted"|
balanced_accuracy|平衡准确度是每个类的召回率算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
f1_score_macro|F1 评分是精度和召回率的调和平均值。 Macro 是每个类的 F1 评分算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="macro"|
f1_score_micro|F1 评分是精度和召回率的调和平均值。 通过统计真报率、漏报率和误报率总值来全局计算 Micro。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="micro"|
f1_score_weighted|F1 评分是精度和召回率的调和平均值。 按每个类的 F1 评分类频率计算的加权平均值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="weighted"|
log_loss|这是（多项式） 逻辑回归及其扩展（例如神经网络）中使用的损失函数，在给定概率分类器的预测的情况下，定义为真实标签的负对数可能性。 对于在 {0,1} 中包含真实标签 yt，且包含 yt=1 的估计概率 yp 的单个样本，对数损失为 -log P(yt&#124;yp) = -(yt log(yp) + (1 - yt) log(1 - yp))。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html)|无|
norm_macro_recall|规范化宏召回率是已规范化的宏召回率，因此，随机性能的评分为 0，完美性能的评分为 1。 可以通过公式 norm_macro_recall := (recall_score_macro - R)/(1 - R) 来计算此值，其中，R 是随机预测的 recall_score_macro 预期值（例如，对于二元分类，R=0.5；对于 C 类分类问题，R=(1/C)）。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average = "macro" |
precision_score_macro|Precision 是正确标记的积极预测元素的百分比。 Macro 是每个类的精度算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="macro"|
precision_score_micro|Precision 是正确标记的积极预测元素的百分比。 通过统计真报率和误报率总值来全局计算 Micro。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="micro"|
precision_score_weighted|Precision 是正确标记的积极预测元素的百分比。 Weighted 是每个类的精度算术平均值，按每个类中的真实实例数加权。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="weighted"|
recall_score_macro|Recall 是特定类的正确标记元素的百分比。 Macro 是每个类的召回率算术平均值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
recall_score_micro|Recall 是特定类的正确标记元素的百分比。 通过统计真报率、漏报率和误报率总值来全局计算 Micro|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="micro"|
recall_score_weighted|Recall 是特定类的正确标记元素的百分比。 Weighted 是每个类的召回率算术平均值，按每个类中的真实实例数加权。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="weighted"|
weighted_accuracy|加权准确度是当分配给每个示例的权重等于该示例的真实类中的真实实例比例时的准确度。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html)|sample_weight 是等于目标中每个元素的该类比例的向量|

### <a name="binary-vs-multiclass-metrics"></a>二元分类指标与多类指标

AutoML 不区分二元分类指标与多类指标。 不管数据集有两个类还是两个以上的类，都会报告相同的验证指标。 但是，某些指标旨在用于多类分类。 正如你所期望的那样，这些指标在应用于二元分类数据集时不会将任何类视为 `true` 类。 明确用于多类的指标以 `micro`、`macro` 或 `weighted` 为后缀。 示例包括 `average_precision_score`、`f1_score`、`precision_score`、`recall_score`、`AUC`。

例如，多类平均召回率（`micro`、`macro` 或 `weighted`）不按 `tp / (tp + fn)` 计算召回率，而是对二进制分类数据集的两个类进行平均。 这相当于分别计算 `true` 类和 `false` 类的召回率，然后取二者的平均值。

## <a name="confusion-matrix"></a>混淆矩阵

混淆矩阵描述了分类模型的性能。 每一行显示真实类的实例或数据集中的实际类，每一列表示模型预测的类的实例。 

对于每个混淆矩阵，自动化 ML 将显示每个预测标签（列）相对于真实标签（行）的频率。 颜色越暗，该特定矩阵部分中的计数越大。 

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？

混淆矩阵将数据集的实际值与模型提供的预测值进行比较。 因此，如果机器学习模型的大部分值沿对角线分布（表示模型预测了正确的值），则该模型具有较高的准确度。 如果模型中的类不平衡，混淆矩阵可帮助检测有偏差的模型。

#### <a name="example-1-a-classification-model-with-poor-accuracy"></a>示例 1：准确度较差的分类模型
![准确度较差的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-confusion-matrix1.png)

#### <a name="example-2-a-classification-model-with-high-accuracy"></a>示例 2：准确度较高的分类模型 
![准确度较高的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-confusion-matrix2.png)

##### <a name="example-3-a-classification-model-with-high-accuracy-and-high-bias-in-model-predictions"></a>示例 3：准确度较高且模型预测偏差较高的分类模型
![准确度较高且模型预测偏差较高的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-biased-model.png)

<a name="precision-recall-chart"></a>

## <a name="precision-recall-chart"></a>精度-召回率图表

精准率-召回率曲线显示模型的精准率与召回率之间的关系。 术语“精准率”表示模型正确标记所有实例的能力。 “召回率”表示分类器查找特定标签的所有实例的能力。

使用此图表可以比较每个模型的精度-召回率曲线，以确定哪个模型的精度与召回率关系可接受，可以解决特定的业务问题。 此图表显示宏观平均精度-召回率、微观平均精度-召回率，以及与模型的所有类关联的精度-召回率。 

“宏观平均”将单独计算每个类的指标，然后取平均值，并同等处理所有类。 然而，“微观平均”将聚合所有类的贡献来计算平均值。 如果数据集中存在类不平衡的情况，则最好是使用微观平均。

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？
根据业务问题的目标，理想的精准率-召回率曲线可能各不相同。 

##### <a name="example-1-a-classification-model-with-low-precision-and-low-recall"></a>示例 1：精准率和召回率较低的分类模型
![精准率和召回率较低的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-precision-recall1.png)

##### <a name="example-2-a-classification-model-with-100-precision-and-100-recall"></a>示例 2：精准率和召回率大约为 100% 的分类模型 
![精准率和召回率较高的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-precision-recall2.png)
<a name="roc"></a>
## <a name="roc-chart"></a>ROC 图

接收方操作特征 (ROC) 是特定模型的正确分类标签与错误分类标签的对比图。 在类失衡严重的情况下基于数据集训练模型时，ROC 曲线提供的信息可能较少，因为多数类可能会掩盖少数类的贡献。

可以按照正确分类的样本的比例将该 ROC 图下的区域可视化。 ROC 图的高级用户可能会查看曲线下区域之外的区域，直观了解以分类阈值或决策边界函数形式表示的真正率和假正率。

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？
最佳模型是 ROC 曲线接近左上角，即真正率为 100%，假正率为 0%。 随机模型将显示为一条从左下角到右上角的平直线。 比随机模型更差的模型会显示在 y=x 这条线的下方。

#### <a name="example-1-a-classification-model-with-low-true-labels-and-high-false-labels"></a>示例 1：真报标签较少且误报标签较多的分类模型
![真报标签较少且误报标签较多的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-roc-1.png)

#### <a name="example-2-a-classification-model-with-high-true-labels-and-low-false-labels"></a>示例 2：真报标签较多且误报标签较少的分类模型
![真报标签较多且误报标签较少的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-roc-2.png)
<a name="lift-curve"></a>

## <a name="lift-chart"></a>提升图

提升图评估分类模型的性能。 提升图显示某个模型的表现优于随机模型的次数。 这里提供的是一个相对表现（考虑到类的数量越多，分类越困难）。 与具有两个类的数据集相比，随机模型对具有 10 个类的数据集中的样本进行预测时，错误率更高。

可将使用 Azure 机器学习自动生成的模型的提升与基线（随机模型）进行比较，以查看该特定模型的值增益情况。

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？

提升曲线越高（即模型在基线之上越高），表示模型性能更好。 

#### <a name="example-1-a-classification-model-that-performs-poorly-compared-to-a-random-selection-model"></a>示例 1：表现比随机选择模型差的分类模型
![比随机选择模型更差的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-lift-curve1.png)

#### <a name="example-2-a-classification-model-that-performs-better-than-a-random-selection-model"></a>示例 2：表现比随机选择模型更好的分类模型
![表现更好的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-lift-curve2.png)
<a name="gains-curve"></a>

## <a name="cumulative-gains-chart"></a>累积增益图

累积增益图按数据的每个部分评估分类模型的表现。 该图按数据集的每个百分位，显示与始终不正确的模型相比，已进行准确分类的额外样本数。 此信息提供了查看随附提升图中的结果的另一种方式。

借助累积增益图，可使用与模型所需增益相对应的百分比来选择分类截止值。 可将累积增益图与基线（不正确的模型）进行比较，查看在每个置信度百分比处正确分类的样本的百分比。

#### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？

与提升图类似，累积增益曲线在基准之上越高，模型的性能越好。 此外，累积增益曲线越接近图表的左上角，模型相对于基线获得的增益越大。 

##### <a name="example-1-a-classification-model-with-minimal-gain"></a>示例 1：增益极低的分类模型
![增益极低的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-gains-curve2.png)

##### <a name="example-2-a-classification-model-with-significant-gain"></a>示例 2：增益很高的分类模型
![增益很高的分类模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-gains-curve1.png)

<a name="calibration-plot"></a>

## <a name="calibration-chart"></a>校准图

校准图显示了预测模型的置信度。 它通过显示预测概率和实际概率之间的关系来实现，其中“概率”表示一个特定实例属于某个标签的可能性。

对于所有分类问题，可以查看微观平均、宏观平均以及给定预测模型中每个类的校准行。

“宏观平均”将单独计算每个类的指标，然后取平均值，并同等处理所有类。 然而，“微观平均”将聚合所有类的贡献来计算平均值。 

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？
进行了适当校准的模型会与 y=x 这条线吻合，会正确预测样本属于每个类的概率。 置信度过高的模型在预测接近零和一的概率时会出现高估的情况，但很少出现无法确定每个样本的类的情况。

#### <a name="example-1-a-well-calibrated-model"></a>示例 1：适当校准的模型
![ 适当校准的模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-calib-curve1.png)

#### <a name="example-2-an-over-confident-model"></a>示例 2：置信度过高的模型
![置信度过高的模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-calib-curve2.png)


<a name="regression"></a> 

## <a name="regression-performance-metrics"></a>回归性能指标

下表总结了 AutoML 针对为试验生成的每个回归模型或预测模型计算的模型性能指标。 

|指标|说明|计算|其他参数
--|--|--|--|
explained_variance|解释方差是数学模型计算给定数据集的方差时遵循的比例。 它是原始数据方差与误差方差之间的递减百分比。 当误差的平均值为 0 时，它等于确定系数（请参见下面的 r2_score）。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.explained_variance_score.html)|无|
r2_score|R^2 是确定系数或平方误差与输出平均值的基线模型相比减少的百分比。 |[计算](https://scikit-learn.org/0.16/modules/generated/sklearn.metrics.r2_score.html)|无|
spearman_correlation|斯皮尔曼相关是两个数据集之间的关系单一性的非参数测量法。 与皮尔逊相关不同，斯皮尔曼相关不假设两个数据集呈正态分布。 与其他相关系数一样，此参数在 -1 和 +1 之间变化，0 表示不相关。 -1 或 +1 相关表示确切的单一关系。 正相关表示 y 随着 x 的递增而递增。 负相关表示 y 随着 x 的递增而递减。|[计算](https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.stats.spearmanr.html)|无|
mean_absolute_error|平均绝对误差是目标与预测之间的差的预期绝对值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|无|
normalized_mean_absolute_error|规范化平均绝对误差是平均绝对误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|除以数据范围|
median_absolute_error|平均绝对误差是目标与预测之间的所有绝对差的中间值。 此损失值可靠地反映离群值。|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|无|
normalized_median_absolute_error|规范化中间绝对误差是中间绝对误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|除以数据范围|
root_mean_squared_error|均方根误差是目标与预测之间的预期平方差的平方根|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|无|
normalized_root_mean_squared_error|规范化均方根误差是均方根误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|除以数据范围|
root_mean_squared_log_error|均方根对数误差是预期平方对数误差的平方根|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|无|
normalized_root_mean_squared_log_error|规范化均方根对数误差指均方根对数误差除以数据范围后的值|[计算](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|除以数据范围|

<a name="pvt"></a>

## <a name="predicted-vs-true-chart"></a> 预测值与真实值图

预测与“真实”显示回归问题的预测值与其相关真实值之间的关系。 

每次运行后，可以查看每个回归模型的预测与真实图形。 为了保护数据隐私，值已装箱在一起，每个箱的大小在图表区域的下半部分显示为条形图。 可将预测模型（带有浅色阴影，其中显示了误差边际）与模型的理想值进行比较。

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？
可以使用此图形来衡量模型的性能，因为预测值与 y=x 行越接近，预测模型的准确度就越高。

#### <a name="example-1-a-classification-model-with-low-accuracy"></a>示例 1：准确度较低的分类模型
![预测准确度较低的回归模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression1.png)

#### <a name="example-2-a-regression-model-with-high-accuracy"></a>示例 2：准确度较高的回归模型 
![预测准确度较高的回归模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression2.png)

<a name="histo"></a> 

## <a name="histogram-of-residuals-chart"></a> 残差直方图

自动化 ML 自动提供残差图来显示回归模型预测中的误差分布。 残差是指预测与实际值之间的差 (`y_pred - y_true`)。 

### <a name="what-does-a-good-model-look-like"></a>良好的模型是怎样的？
若要显示偏差较小的误差边际，应该将残差直方图绘制成以零为中心的钟形曲线。

#### <a name="example-1-a-regression-model-with-bias-in-its-errors"></a>示例 1：误差中带有偏差的回归模型
![误差中带有偏差的 SA 回归模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression3.png)

#### <a name="example-2-a-regression-model-with-more-even-distribution-of-errors"></a>示例 2：误差较均匀分布的回归模型
![误差较均匀分布的回归模型](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression4.png)

<a name="explain-model"></a>

## <a name="model-interpretability-and-feature-importance"></a> 模型可解释性和特征重要性
自动化 ML 为运行提供机器学习可解释性仪表板。

若要详细了解如何启用可解释性特征，请参阅[可解释性：自动化机器学习中的模型说明](how-to-machine-learning-interpretability-automl.md)。

> [!NOTE]
> 解释客户端目前不支持 ForecastTCN 模型。 如果此模型作为最佳模型返回，则不会返回解释仪表板，并且不支持按需解释运行。

## <a name="next-steps"></a>后续步骤

+ 详细了解 Azure 机器学习中的[自动化机器学习](concept-automated-ml.md)。
+ 请参阅[自动化机器学习模型解释](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model)示例笔记本。
