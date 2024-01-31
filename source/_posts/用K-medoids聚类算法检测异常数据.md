---
title: 用K-medoids聚类算法检测异常数据
date: 2024-01-31 03:00:00
categories: 机器学习
---

# 用K-medoids聚类算法检测异常数据

## 操作过程

## 操作代码

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
print("Distances:", distances)

# 设置异常点的阈值
threshold = 180

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
