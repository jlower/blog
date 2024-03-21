---
title: 用集成学习Ensemble Learning优化检测异常数据的模型
date: 2024-03-20 03:00:00
categories: 机器学习
---

## 集成学习 Ensemble Learning 介绍

> [几何直观地 理解集成学习 Ensemble Learning 的四大类型](https://zhuanlan.zhihu.com/p/494333159)
> [集成学习 Ensemble Learning 简析](https://zhuanlan.zhihu.com/p/682354878)
> [集成学习 Ensemble Learning 简析](https://www.zhihu.com/question/29036379/answers/updated)

集成学习(Ensemble Learning)是指：在机器学习中，模型单独运行时可能表现不佳，但将多个模型组合起来时就会变得更强大，这种多个基础模型的组合称为集成模型或集成学习。

- 基础模型：差异越大，那么它们组合后的集成学习效果越好
- 集成学习：也可以理解为，各个基础模型都是处理不同问题的专家，组合在一起就可以取得更好的模型

集成学习的常见4种类型：

- Bagging(Bootstrapped Aggregation)：装袋算法
- Booting：提升算法，可将弱学习器提升为强学习器的算法
- Stacking：堆叠法
- Cascading：级联法

> 在Bagging中：基础模型为 低偏差，高方差；如深度特别深的 决策树
> 在Boosting中：基础模型为 高偏差，低方差；如深度特别浅的 决策树(如：深度为2，3)

## 集成学习 Bagging 中的 随机森林(Random forest)算法

> [介绍，并展示对Iris数据集分类](https://blog.csdn.net/Code_and516/article/details/131305795)

集成学习是一种强大的机器学习策略，它结合了多个模型以改善整体性能和稳健性。
随机森林(Random Forest)算法是集成学习中Bagging（Bootstrap Aggregating）方法的经典例子。
随机森林的基本概念:

- 随机森林由多个决策树组成，这些决策树作为算法名字中的“森林”。每个决策树都是在数据集的不同随机子样本上训练得到的，采取的是有放回抽样（即bootstrap抽样）。在对新样本进行预测时，随机森林算法会考虑所有单个决策树的预测结果，并通过投票（分类问题）或平均（回归问题）的方式进行决策。

随机森林的关键要点:

1. 构造决策树的随机性：
    - 在构建每棵树时，使用随机抽样的方式从原始数据集中抽取训练数据（bootstrap样本）。
    - 在选择分裂特征时，不再考虑所有特征，而是随机选择一部分特征进行最佳拆分点的计算。
1. 决策树的多样性：
    - 每棵树在不同的样本集上训练，保证了森林中的树具有多样性。
    - 特征的随机选择也增加了决策树之间的差异性，从而提高整个模型的泛化能力。
1. 减少过拟合：
    - 个体树的随机性和多样性帮助减少模型的过拟合问题。
    - 即使单棵树对某些样本过度拟合，整体随机森林模型仍然能够保持较高的预测准确性。
1. 适用性广泛:
    - 随机森林算法适用于分类和回归任务。
    - 模型对数据集中的异常值和噪声具有很高的容忍度。
1. 特征重要性评估：
    - 随机森林能够评估不同特征在决策过程中的重要性，这是通过观察特征在树中的分裂能力得出的。

算法过程：

1. 随机选择N个样本（选择的样本数量通常与原始数据集大小相同）来构建一个bootstrap样本集。
1. 如果特征总数为M，则在构建树的每个节点时随机选择m个特征（其中m << M），通常m的值为sqrt(M)，对于回归问题可能会选择M/3。
1. 在这些特征中找到最佳分裂点来分裂节点，直到每个叶节点的样本数量小于设定的阈值，或者每个叶节点的样本都属于同一类别。
1. 重复以上步骤来构建足够数量的决策树。

最后的预测过程：

- 分类问题： 对于一个新的样本，每棵树给出一个预测结果。最后的类别预测由大多数树投票决定。
- 回归问题： 通过所有树对新样本的预测结果取平均值作为最终的预测。

随机森林是一个简单、易于实现且在许多领域都非常高效的算法。与单一的决策树相比，随机森林往往能够获得更好的性能，而且由于每棵树具有很高的多样性，减少了过拟合的风险。

### 在随机森林算法中，n_estimators 和 random_state 是两个重要的参数，可以通过调整它们来优化模型的性能

- n_estimators：这个参数指定了随机森林中包含的决策树的数量。增加 n_estimators 可以增加模型的复杂度和稳定性，通常会提高模型的性能，但也会增加训练时间。可以通过交叉验证等方法来选择合适的 n_estimators 值。

- random_state：这个参数用于控制随机性，可以指定一个整数值作为随机数生成器的种子，以确保结果的可重复性。在模型调优过程中，可以固定 random_state，这样每次运行时得到的结果都是相同的，有助于稳定模型性能的评估。

例如，在使用 GridSearchCV 进行网格搜索调优时，可以指定不同的 n_estimators 和 random_state 值，然后通过交叉验证选择最佳的参数组合。示例代码如下：

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'n_estimators': [50, 100, 150],
    'random_state': [0, 1, 2]
}

rf = RandomForestClassifier()

grid_search = GridSearchCV(rf, param_grid, cv=3, n_jobs=-1)
grid_search.fit(X_train, y_train)

print("Best parameters:", grid_search.best_params_)
```

### 在二分类问题中，probs 是指模型预测样本属于正例的概率，而 threshold 则是用来判断样本是否为正例的阈值。一般情况下，threshold 的取值范围是 0 到 1 之间

- 默认情况下，当 probs 大于 0.5 时，样本被预测为正例（1），小于等于 0.5 时被预测为负例（0）。但是，threshold 的选择可以根据具体问题和模型调优的需求来调整。较低的 threshold 可能会增加正例的识别率，但也会增加误报率；而较高的 threshold 则可能会减少误报率，但会降低正例的识别率。
- 在实际应用中，可以通过绘制ROC曲线和计算AUC值来选择最优的 threshold。ROC曲线可以帮助我们理解在不同 threshold 下模型的表现情况，AUC值则可以作为评价模型性能的一个指标。

#### 绘制ROC曲线并计算AUC值(ROC曲线下面积)来选择最优的threshold

要绘制ROC曲线，可以使用scikit-learn中的roc_curve函数计算ROC曲线的真正例率（true positive rate，又称为召回率）和假正例率（false positive rate），然后使用matplotlib库绘制曲线。
在这个示例中，y_true是真实标签，y_score是模型预测的概率。roc_curve函数会返回三个数组：假正例率（fpr）、真正例率（tpr）和阈值（thresholds）。auc函数用于计算曲线下的面积，即AUC值。最后，使用matplotlib库绘制ROC曲线图，并标出AUC值。

```python
from sklearn.metrics import roc_curve, auc
import matplotlib.pyplot as plt

# 假设y_true是真实标签，y_score是模型预测的概率
fpr, tpr, thresholds = roc_curve(y_true, y_score)

# 计算AUC值
roc_auc = auc(fpr, tpr)

# 绘制ROC曲线
plt.figure()
lw = 2
plt.plot(fpr, tpr, color='darkorange',
         lw=lw, label='ROC curve (area = %0.2f)' % roc_auc)
plt.plot([0, 1], [0, 1], color='navy', lw=lw, linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver Operating Characteristic')
plt.legend(loc="lower right")
plt.show()
```

计算AUC（Area Under the Curve）值可以使用scikit-learn中的roc_auc_score函数。这个函数接受真实标签和预测概率作为输入，并返回AUC值。
这段代码会计算真实标签y_true和模型预测的概率y_score之间的ROC曲线下面积（AUC值）。

```python
from sklearn.metrics import roc_auc_score

# 假设y_true是真实标签，y_score是模型预测的概率
auc_score = roc_auc_score(y_true, y_score)
print("AUC Score:", auc_score)
```

### 代码，批量读取文件夹下每个文件分别训练，将结果保存到文件里，并可视化

```python
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import confusion_matrix, accuracy_score, precision_score, recall_score, f1_score

# 定义存储汇总信息的列表
summary_results = []

# 定义函数
def process_file(file_path):
    # 读取数据
    data = pd.read_csv(file_path)
    total_samples = len(data)
    
    # 随机森林算法识别异常值
    clf = RandomForestClassifier(n_estimators=100, random_state=0)
    clf.fit(data['value'].values.reshape(-1, 1), data['anomaly'])
    probs = clf.predict_proba(data['value'].values.reshape(-1, 1))[:, 1]
    threshold = 0.5
    outliers = np.where(probs > threshold, 1, 0)
    
    # 评估模型
    conf_matrix = confusion_matrix(data['anomaly'], outliers)
    accuracy = accuracy_score(data['anomaly'], outliers)
    precision = precision_score(data['anomaly'], outliers)
    recall = recall_score(data['anomaly'], outliers)
    f1 = f1_score(data['anomaly'], outliers)
    tp, fp, tn, fn = conf_matrix.ravel()
    
    # 记录汇总信息
    summary_results.append({
        'File': os.path.basename(file_path),
        'Accuracy': accuracy,
        'Precision': precision,
        'Recall': recall,
        'F1 Score': f1,
        'TP': tp,
        'FP': fp,
        'TN': tn,
        'FN': fn,
        'Total Samples': total_samples
    })
    
    # 可视化数据
    plt.figure(figsize=(10, 6))
    plt.scatter(data.index, data['value'], c=outliers, cmap='coolwarm')
    plt.title('Time Series Data with Anomalies')
    plt.xlabel('Index')
    plt.ylabel('Value')
    plt.colorbar(label='Outlier')
    output_path = os.path.join(output_folder_path, 'each_files')
    output_file_path = os.path.join(output_path, os.path.basename(file_path))
    os.makedirs(output_path, exist_ok=True)
    plt.savefig(output_file_path.replace('.csv', '_plot.png')) # TODO 不保存图片要注释
    plt.show()
    plt.close()

# 指定输入文件夹路径和输出文件夹路径
input_folder_path = '/kaggle/input/ncep-test-2-anomaly01'
output_folder_path = '/kaggle/working/output'
os.makedirs(output_folder_path, exist_ok=True)

# 遍历文件夹
for file_name in os.listdir(input_folder_path):
    file_path = os.path.join(input_folder_path, file_name)
    process_file(file_path)

# 将关键指标汇总并输出到文件
summary_df = pd.DataFrame(summary_results)
summary_df.to_csv(os.path.join(output_folder_path, './summary_results.csv'), index=False)

# 可视化为柱状图
plt.figure(figsize=(18, 10))

plt.subplot(2, 2, 1)
sns.barplot(x='File', y='Accuracy', data=summary_df)
plt.xticks(rotation=45, ha='right')
plt.title('Accuracy for Each File')
plt.xlabel('File')
plt.ylabel('Accuracy')

plt.subplot(2, 2, 2)
sns.barplot(x='File', y='Precision', data=summary_df)
plt.xticks(rotation=45, ha='right')
plt.title('Precision for Each File')
plt.xlabel('File')
plt.ylabel('Precision')

plt.subplot(2, 2, 3)
sns.barplot(x='File', y='Recall', data=summary_df)
plt.xticks(rotation=45, ha='right')
plt.title('Recall for Each File')
plt.xlabel('File')
plt.ylabel('Recall')

plt.subplot(2, 2, 4)
sns.barplot(x='File', y='F1 Score', data=summary_df)
plt.xticks(rotation=45, ha='right')
plt.title('F1 Score for Each File')
plt.xlabel('File')
plt.ylabel('F1 Score')

plt.tight_layout()
plt.savefig(os.path.join(output_folder_path, './summary_result_metrics_barplots.png')) # TODO 不保存图片要注释
plt.show()
plt.close()
```
