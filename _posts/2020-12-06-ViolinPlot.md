---
layout: post
title:  "Python绘制小提琴图"
tags:   Python Utils
date:   2020-12-06 10:00:10 +0800
categories: [Utils]
---

- Source Code

  ```python
  #加载需要的模块
  import warnings
  warnings.filterwarnings('ignore')
  import pandas as pd
  import numpy as np
  import matplotlib.pyplot as plt
  import seaborn as sns
  
  #导入数据
  Train_data = pd.read_csv('Star.csv')
  # Train_data['gearbox'].value_counts() #对分类变量的类别进行计数
  #后面将研究不同类型的'gearbox'对应'price'的差异
  # x=Train_data['gearbox']
  # y=Train_data['price'] #在原数据集中，'price'为目标变量
  #绘制小提琴图
  sns.violinplot(x='Type',y='Num',data=Train_data)
  plt.yscale('log')
  plt.show()
  #在sns.violinplot中，x是类别变量，y是数值型变量，data用于指定数据集
  ```

  
