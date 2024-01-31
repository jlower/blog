---
title: 用K-medoids聚类算法检测异常数据
date: 2024-01-31 03:00:00
categories: 机器学习
---

# 用K-medoids聚类算法检测异常数据

## 操作过程

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
## 操作代码

### 确定聚类的簇数K

> 可以通过尝试不同的K值并使用合适的评估指标（如轮廓系数、肘部法则等）来选择最佳的K值。

```python
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
