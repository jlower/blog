---
title: 用K-medoids聚类算法检测异常数据
date: 2024-01-31 03:00:00
categories: 机器学习
---

## K-medoids聚类算法

### K-Medoids 介绍

> K-Medoids 是 K-Means 的一种改进版本，用于聚类分析。与 K-Means 不同，K-Medoids 选择实际数据点作为聚类的中心，而不是取均值。这样的中心点被称为“medoids”。

#### K-Medoids 算法步骤

1. **选择初始中心点：** 从数据集中随机选择 k 个数据点作为初始中心点。
1. **分配数据点到最近的中心点：** 对每个数据点，计算其与每个中心点的距离，并将其分配到距离最近的中心点所属的簇。
1. **更新中心点（Medoids）：** 对于每个簇，选择其中一个数据点替代当前的中心点，以最小化该簇中所有数据点到新中心点的距离之和。
1. **重复步骤 2 和 3：** 重复执行分配和更新步骤，直至簇的分配不再改变或达到预定的迭代次数。

#### 优点

- **对异常值较为鲁棒：** 由于中心点是实际的数据点，K-Medoids 对异常值较为敏感，这使得它对于异常值的存在有一定的鲁棒性。
- **适用于离散型数据：** 由于 K-Medoids 使用实际数据点作为中心，因此它适用于离散型数据。

### K-Medoids 时间复杂度

> K-Medoids 算法的时间复杂度取决于其迭代次数和每次迭代的计算成本。总体而言，K-Medoids 是一个迭代的优化算法，其主要步骤包括选择中心点（medoids）和更新簇的分配，直至收敛。

1. **选择中心点（初始化）：** K-Medoids 的初始步骤涉及选择 k 个中心点，这个过程的时间复杂度为 O(kn)，其中 n 是数据点的数量。通常，这一步的计算成本取决于选择初始中心点的策略，可能采用随机选择或者其他启发式方法。
1. **更新簇的分配：** K-Medoids 的主要计算成本在于不断迭代地更新每个点到最近中心点的分配，然后根据这些分配更新中心点。这一步的时间复杂度通常为 O(iter \* k \* n \* d)，其中 iter 是迭代次数，k 是簇的数量，n 是数据点的数量，d 是数据点的维度。
1. **停止条件：** K-Medoids 需要达到收敛条件，通常是簇的分配不再发生变化。停止条件的检查可能涉及对所有数据点的遍历，时间复杂度为 O(n)。

> 综合考虑，K-Medoids 算法的总体时间复杂度通常在 O(iter \* k \* n \* d) 的数量级，其中 iter、k、n、d 分别是迭代次数、簇的数量、数据点的数量和数据点的维度。

## 一维数据操作

### 确定聚类的簇数K

> 可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。

#### 轮廓系数(Silhouette Coefficient)

> Silhouette Coefficient 是一个衡量聚类效果的指标，其值在 -1 到 1 之间。具体而言，对于每个数据点，Silhouette Coefficient 考虑了该点与同簇其他点的相似度和该点与最近的其他簇的点的不相似度。一个高的 Silhouette Coefficient 表示簇内的样本相似度高且簇间的样本相似度低。  
> 在曲线中，轮廓系数越接近1表示聚类效果越好。可以选择具有最高轮廓系数的K值。

#### 肘部法则(Elbow Method)

> 肘部法则通过绘制不同K值下的聚类模型的损失函数（如簇内平方和）的图形来帮助选择合适的K值。K的选择通常发生在损失函数开始减缓的"肘部"位置。  
> 在图形中，你会看到一个肘部，选择这个肘部对应的K值作为最优的聚类数目。需要注意的是，这个方法并不总是完全明显，有时可能需要主观判断。

## 一维数据操作代码

### 数据预处理

```python
# 提取'value'列作为特征
data = dataframe_total['value'].values.reshape(-1, 1)
```

### 确定K值

> 可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。

```python
# 提取'value'列作为特征
data = dataframe_total['value'].values.reshape(-1, 1)

from sklearn_extra.cluster import KMedoids
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# 选择可能的K值范围
k_values = range(2, 20)

# 轮廓系数(Silhouette Coefficient)
# 计算每个K值对应的轮廓系数
silhouette_scores = []
for k in k_values:
    kmedoids = KMedoids(n_clusters=k, random_state=3, init='k-medoids++')
    cluster_labels = kmedoids.fit_predict(data)
    silhouette_avg = silhouette_score(data, cluster_labels)
    silhouette_scores.append(silhouette_avg)

# 绘制轮廓系数曲线
plt.plot(k_values, silhouette_scores, marker='o')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Silhouette Coefficient')
plt.title('Silhouette Coefficient for Different K Values')
plt.show()

# 肘部法则(Elbow Method)
# 计算每个K值对应的损失函数值
inertia_values = []
for k in k_values:
    kmedoids = KMedoids(n_clusters=k, random_state=3, init='k-medoids++')
    kmedoids.fit(data)
    inertia_values.append(kmedoids.inertia_)

# 绘制损失函数值曲线
plt.plot(k_values, inertia_values, marker='o')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Inertia (Within-Cluster Sum of Squares)')
plt.title('Elbow Method for Optimal K')
plt.show()
```

### 训练、执行并进行分析可视化

```python
# 提取'value'列作为特征
data = dataframe_total['value'].values.reshape(-1, 1)

from sklearn_extra.cluster import KMedoids
from sklearn.metrics import pairwise_distances_argmin_min, confusion_matrix
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score
import time

# 选择K值
k = 4

t0 = time.time()  # 计算训练数据消耗的时间

# 使用 K-medoids 进行聚类
kmedoids = KMedoids(n_clusters=k, random_state=42)

# fit(data) : 这个方法用于对模型进行拟合（训练），但它并不返回预测的聚类标签。使用fit方法训练模型后，你可以使用模型的其他属性，如cluster_centers_，来获取聚类的中心点等信息。
kmedoids.fit(data)
cluster_labels = kmedoids.labels_
medoid_indices = kmedoids.medoid_indices_
cluster_centers = data[medoid_indices]

# fit_predict(data) : 这个方法用于对模型进行拟合（训练）并返回每个样本的聚类标签。这是一个方便的方法，因为它在一步中完成了训练和预测。
# cluster_labels = kmedoids.fit_predict(data)

# 计算每个数据点到其分配的medoid的距离，将距离超过一定阈值的点视为异常点。
# 检测异常值，可以使用距离度量，例如欧氏距离
distances = np.linalg.norm(data - data[medoid_indices[cluster_labels]], axis=1)
# print("Distances:", distances)

# 设置异常点的阈值
threshold = 0.8

# 标记异常点
outliers_kmedoids = dataframe_total.index[distances > threshold]

# 输出结果
print("Medoids:", kmedoids.cluster_centers_.flatten())
print("Outliers:", outliers_kmedoids)

KMedoids_batch = time.time() - t0  # 使用K-medoids训练数据消耗的时间
print("K-medoids算法模型训练消耗时间:%.4fs" % KMedoids_batch)

# 与'anomaly'列进行比较
# true_labels列表中 0为正常 1为粗差
# .values 转换成 numpy.ndarray 列表
true_labels = dataframe_total['anomaly'].values

# 在混淆矩阵中,通常约定: 0 表示负例 (Negative) ; 1 表示正例 (Positive)
# 通常 负例是“正常”或“非异常” ; 正例是“异常”或“错误”

# 如果要处理 true_labels 把他换成混淆矩阵中约定的格式 -> 将true_labels列表中的 0变为1 1变为0
# 使用 np.array() 把 普通列表 转换为 numpy.ndarray 列表:
# true_labels_inverted = np.array([0 if label==1 else 1 for label in true_labels]) 
# 或者使用:
# true_labels_inverted = 1 - true_labels

# 计算混淆矩阵
conf_matrix = confusion_matrix(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])

# 计算指标
accuracy = accuracy_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
precision = precision_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
recall = recall_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
f1 = f1_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])

# 输出结果
print("Confusion Matrix:\n", conf_matrix)
print("True Negative (TN):", conf_matrix[0, 0])
print("False Positive (FP):", conf_matrix[0, 1])
print("False Negative (FN):", conf_matrix[1, 0])
print("True Positive (TP):", conf_matrix[1, 1])
print("Accuracy:", accuracy)
print("Precision:", precision)
print("Recall:", recall)
print("F1 Score:", f1)

# 可视化
plt.figure(figsize=(12, 6))

plt.subplot(1, 2, 1)
sns.scatterplot(x=dataframe_total.index, y=dataframe_total['value'], hue=true_labels, palette='viridis', marker='o',
                edgecolor='k')
# plt.scatter(outliers_kmedoids, dataframe_total.loc[outliers_kmedoids, 'value'], c='red', marker='x', s=100,
#             label='Outliers (K-medoids)')
# plt.title('Actual Anomalies vs K-medoids Outliers')
plt.title('Actual Anomalies')
plt.xlabel('Index')
plt.ylabel('Value')
plt.legend()

plt.subplot(1, 2, 2)
df_plot = pd.DataFrame({'Index': dataframe_total.index, 'Value': dataframe_total['value'], 'Cluster': cluster_labels})
sns.scatterplot(x='Index', y='Value', hue='Cluster', palette='viridis', marker='o', edgecolor='k', data=df_plot)
plt.scatter(outliers_kmedoids, dataframe_total.loc[outliers_kmedoids, 'value'], c='red', marker='x', s=100,
            label='Outliers (K-medoids)')
plt.scatter([np.mean(dataframe_total['value'])] * k, kmedoids.cluster_centers_.flatten(), c='blue', marker='X', s=200,
            label='Medoids')
plt.title('K-medoids Clustering and Outlier Detection')
plt.xlabel('Index')
plt.ylabel('Value')
plt.legend()

plt.tight_layout()
plt.show()
```

### 批量读取文件夹下每个文件,分别训练;将结果保存到文件里,且可视化

```python
import os
import pandas as pd
import numpy as np
from sklearn_extra.cluster import KMedoids
from sklearn.metrics import confusion_matrix, accuracy_score, precision_score, recall_score, f1_score
import matplotlib.pyplot as plt
import seaborn as sns

# 指定输入文件夹路径和输出文件夹路径
input_folder_path = '/kaggle/input/ncep-test-2-anomaly01'
output_folder_path = '/kaggle/working/output'
os.makedirs(output_folder_path, exist_ok=True)

# 获取输入文件夹下所有文件的文件名
file_names = os.listdir(input_folder_path)

# 存储每个数据集的关键指标
summary_results = []

# 遍历每个文件
for file_name in file_names:
    # 构建文件的完整输入路径
    input_file_path = os.path.join(input_folder_path, file_name)

    # 读取文件数据
    dataframe_total = pd.read_csv(input_file_path)

    # 提取 'value' 列作为特征
    data = dataframe_total['value'].values.reshape(-1, 1)

    # 选择 K 值
    k = 3

    # 使用 K-Medoids 进行聚类
    kmedoids = KMedoids(n_clusters=k, random_state=42)
    kmedoids.fit(data)

    # 获取聚类结果和异常点
    cluster_labels = kmedoids.labels_
    medoid_indices = kmedoids.medoid_indices_
    cluster_centers = data[medoid_indices]
    distances = np.linalg.norm(data - data[medoid_indices[cluster_labels]], axis=1)
    threshold = 0.8
    outliers_kmedoids = dataframe_total.index[distances > threshold]

    # 与 'anomaly' 列进行比较
    true_labels = dataframe_total['anomaly'].values

    # 计算混淆矩阵和指标
    conf_matrix = confusion_matrix(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
    accuracy = accuracy_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
    precision = precision_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
    recall = recall_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])
    f1 = f1_score(true_labels, [1 if idx in outliers_kmedoids else 0 for idx in dataframe_total.index])

    # 将关键指标存储到列表中
    summary_results.append({
        'Dataset': file_name,
        'Confusion Matrix': conf_matrix,
        'True Negative (TN)': conf_matrix[0, 0],
        'False Positive (FP):': conf_matrix[0, 1],
        'False Negative (FN)': conf_matrix[1, 0],
        'True Positive (TP)': conf_matrix[1, 1],
        'Accuracy': accuracy,
        'Precision': precision,
        'Recall': recall,
        'F1 Score': f1
    })

    # 构建输出文件的完整输出路径
    output_file_path = os.path.join(output_folder_path, os.path.splitext(file_name)[0] + '_kmedoids_results.txt')
    # print(output_file_path)

    # 输出结果到文件
    with open(output_file_path, 'w') as file:
        file.write("Confusion Matrix:\n{}\n".format(conf_matrix))
        file.write("True Negative (TN): {}\n".format(conf_matrix[0, 0]))
        file.write("False Positive (FP): {}\n".format(conf_matrix[0, 1]))
        file.write("False Negative (FN): {}\n".format(conf_matrix[1, 0]))
        file.write("True Positive (TP): {}\n".format(conf_matrix[1, 1]))
        file.write("Accuracy: {}\n".format(accuracy))
        file.write("Precision: {}\n".format(precision))
        file.write("Recall: {}\n".format(recall))
        file.write("F1 Score: {}\n".format(f1))
        file.write("====================================================\n")
        file.write("Medoids: {}\n".format(kmedoids.cluster_centers_.flatten()))
        file.write("Outliers: {}\n".format(outliers_kmedoids))

# 将关键指标汇总并输出到文件
summary_df = pd.DataFrame(summary_results)
summary_df.to_csv(os.path.join(output_folder_path, '../summary_results.csv'), index=False)

# 输出适应情况总结
print("Summary Results:")
print(summary_df)

# 可视化关键指标
plt.figure(figsize=(10, 6))
sns.barplot(x='Dataset', y='Precision', data=summary_df, palette='viridis')
plt.title('Precision Across Datasets')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 6))
sns.barplot(x='Dataset', y='Recall', data=summary_df, palette='viridis')
plt.title('Recall Across Datasets')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 6))
sns.barplot(x='Dataset', y='F1 Score', data=summary_df, palette='viridis')
plt.title('F1 Score Across Datasets')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 6))
sns.barplot(x='Dataset', y='Accuracy', data=summary_df, palette='viridis')
plt.title('Accuracy Across Datasets')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

## 二维数据操作

### 确定聚类的簇数K

> 可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。

#### 轮廓系数(Silhouette Coefficient)

> Silhouette Coefficient 是一个衡量聚类效果的指标，其值在 -1 到 1 之间。具体而言，对于每个数据点，Silhouette Coefficient 考虑了该点与同簇其他点的相似度和该点与最近的其他簇的点的不相似度。一个高的 Silhouette Coefficient 表示簇内的样本相似度高且簇间的样本相似度低。  
> 在曲线中，轮廓系数越接近1表示聚类效果越好。可以选择具有最高轮廓系数的K值。

![20240131213741](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/20240131213741.png>)

#### 肘部法则(Elbow Method)

> 肘部法则通过绘制不同K值下的聚类模型的损失函数（如簇内平方和）的图形来帮助选择合适的K值。K的选择通常发生在损失函数开始减缓的"肘部"位置。  
> 在图形中，你会看到一个肘部，选择这个肘部对应的K值作为最优的聚类数目。需要注意的是，这个方法并不总是完全明显，有时可能需要主观判断。

![20240131220326](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/20240131220326.png>)

### 训练-聚类分簇并计算找出异常点

#### 聚类分簇的结果

![20240131221333](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/20240131221333.png>)

#### 找出异常点的结果

![20240131221418](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/20240131221418.png>)

## 二维数据操作代码

### 确定聚类的簇数K

> 可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。

```python
# 转换成二维numpy.ndarray列表
data = dataframe_total.iloc[:, :].values

from sklearn_extra.cluster import KMedoids
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt

# 选择可能的K值范围
k_values = range(2, 20)

# 轮廓系数(Silhouette Coefficient)
# 计算每个K值对应的轮廓系数
silhouette_scores = []
for k in k_values:
    kmedoids = KMedoids(n_clusters=k, random_state=3, init='k-medoids++')
    cluster_labels = kmedoids.fit_predict(data)
    silhouette_avg = silhouette_score(data, cluster_labels)
    silhouette_scores.append(silhouette_avg)

# 绘制轮廓系数曲线
plt.plot(k_values, silhouette_scores, marker='o')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Silhouette Coefficient')
plt.title('Silhouette Coefficient for Different K Values')
plt.show()

# 肘部法则(Elbow Method)
# 计算每个K值对应的损失函数值
inertia_values = []
for k in k_values:
    kmedoids = KMedoids(n_clusters=k, random_state=3, init='k-medoids++')
    kmedoids.fit(data)
    inertia_values.append(kmedoids.inertia_)

# 绘制损失函数值曲线
plt.plot(k_values, inertia_values, marker='o')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Inertia (Within-Cluster Sum of Squares)')
plt.title('Elbow Method for Optimal K')
plt.show()
```

### 使用ChatGPT(单文件执行)

```python
from sklearn_extra.cluster import KMedoids
import numpy as np
from sklearn.metrics import pairwise_distances_argmin_min
import matplotlib.pyplot as plt
import time

# 确定聚类的簇数K。可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。
# 选择K值
k = 8

# 定义K-Medoids模型
kmedoids = KMedoids(n_clusters=k, random_state=3, init='k-medoids++')

t0 = time.time()  # 计算训练数据消耗的时间

# 使用K-medoids进行聚类
kmedoids.fit(data)

# 获取簇标签和中心点索引
cluster_labels = kmedoids.labels_
medoid_indices = kmedoids.medoid_indices_
# 获取最终的medoids
medoids = data[kmedoids.medoid_indices_]

# 计算每个数据点到其分配的medoid的距离，将距离超过一定阈值的点视为异常点。
# 检测异常值，可以使用距离度量，例如欧氏距离
distances = np.linalg.norm(data - data[medoid_indices[cluster_labels]], axis=1)
# print("Distances:", distances)

# 设置异常点的阈值
threshold = 190

# 标记异常点
outliers = np.where(distances > threshold)[0]

KMedoids_batch = time.time() - t0  #使用K-medoids训练数据消耗的时间
print("K-medoids算法模型训练消耗时间:%.4fs" % KMedoids_batch)

# 输出结果
print("Medoids:", medoids)
print("Outliers:", outliers)

# 可视化聚类结果
plt.scatter(data[:, 0], data[:, 1], c=kmedoids.labels_, cmap='viridis', marker='o', edgecolors='k')
plt.scatter(medoids[:, 0], medoids[:, 1], c='red', marker='X', s=200, label='Medoids')
plt.title('K-medoids Clustering')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.show()

# 可视化异常点
plt.scatter(data[:, 0], data[:, 1], c='blue', marker='o', edgecolors='k', label='Normal')
plt.scatter(data[outliers, 0], data[outliers, 1], c='red', marker='x', s=100, label='Outliers')
plt.title('Outlier Detection')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.show()
```
