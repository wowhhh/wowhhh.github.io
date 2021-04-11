---
layout: post
title:  "快排的三数取中与三向切分"
tags:   面试 
date:   2021-04-09 9:00:00 +0800
categories: [面试]


---

### QC基础写法

### 三向取中

 pivot 的选取尤为重要，选取时应尽量避免选取序列的最大或最小值做为基准值。则我们可以利用三数取中法来选取基准值。

![image-20210409211638597](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210409211638597.png)

就比如上面的例子，选7为pivot，循环了一圈，就换个2和7，不大行。

### 三向切分

解决相同元素太多的情况

![image-20210409211835833](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210409211835833.png)

### 参考

[袁厨](https://github.com/chefyuan/algorithm-base/blob/main/animation-simulation/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E7%AE%97%E6%B3%95/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F.md)

[三项切分的快排](https://www.jianshu.com/p/d70aeccaee19)