---
layout: post
title:  "Python绘制小提琴图"
tags:   Python Utils
date:   2020-12-06 10:00:10 +0800
categories: [Utils]
---

#### 基本绘制

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

#### 颜色更改

使用RGB与十六进制的转换工具：https://www.sioe.cn/yingyong/yanse-rgb-16/

```python
sns.violinplot(data=logData,color='#9DC4BD')
```

以上述方式指定即可

#### Log Scale

##### 直接指定log scalse

##### 将数据手动转化为log后的数据

 transform the data

###### log & ln

#### 刻度更改

```python
    plt.ylim(ylims)
    plt.yticks(ytiks)
```

#### 设置图像宽度

