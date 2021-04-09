---
layout: post
title:  "华为实习一面的一道算法题"
tags:   面试 
date:   2021-04-09 9:00:00 +0800
categories: [面试]


---

### 题目描述
>给定一个整数序列如[-2, -3, 0, 4, 9] 和 数字k,要求输出序列中最长的连续子序列的长度，满足其和等于k，如未找到，输出0 ， 如k=-1, 输出4,因为最长序列是[-2, -3, 0, 4];  k=1,输出3

### Brute-Force

暴力解法就双重for循环盘他

```java
    public static int max(int[] arr,int k)
    {
        //用来记录最大的长度，默认是0
        int max = 0;
        //i表示当前下标开始，会用来计算，以i下标为开始，循环到+1到+length-i的长度
        for(int i = 0;i<arr.length;i++)
        {
            int sum = arr[i];
            if(sum==k) max =Math.max(max,1);
            //循环
            for(int j = i+1;j<arr.length;j++)
            {
                sum+=arr[j];
                if(sum==k) max =Math.max(max,j-i+1);
            }
        }
        return max;
    }
```



#### Prefix Sum

在我们使用这个方法之前，先看一道Leetcode的题目：[560. 和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

这里所求的是数组中，和为k的子数组的个数。

思路：

>记录每一位的前缀和，并且保存当前前缀和的值出现的次数
>
>遍历前缀和的数组，如果当前map里面包含了此时的前缀和-k的值，就说明我们知道某一个区间的和为k了，这时候就可以把次数获取下来了
>
>![image-20210409205318603](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210409205318603.png)

那么回过头来看我们这个问题，其实整体和上面差不过，只不过上面题目记录的是k的次数，而我们需要统计出现k的时候，此时子数组有多长，从上面的图片可以看出，假设presum的右端是 i，那么map里如果有presum-k，我们就需要把presum-k这个前缀和的下标记录在map中，这个时候直接相减就是j的长度了

方法为：Prefix + HashMap

```java
    //我们就以 -2，-3，0，4，9为例
	public static int correct(int[] arr,int k)
    {
        //定义前缀和数组，这里用的不是下标，1就是1
        int[] prefixNum = new int[arr.length+1];
        //循环计算前缀和
        for(int i = 0;i<arr.length;i++)
        {
            prefixNum[i+1] = prefixNum[i]+arr[i];
        }
        //定义Map，Key为前缀和的值，Value为此前缀和下标（如果前缀和重复了怎么办？？？）
        //猜测：遇到重复的话，我们是需要使用坐标进来小的，因为这样才能使得k进来大
        Map<Integer,Integer> map = new HashMap<Integer, Integer>();
        map.put(0,0);//左边界
        int res = 0;
        //循环前缀和数组里面的每一个值
        for(int i = 0;i<prefixNum.length;i++)
        {
            //计算我们要找的目标前缀和
            int target = prefixNum[i] - k;
            //如果map中包含target就说明了整个数组里面有子数组能够做到其和为k，这个时候需要去除其下标，做差即可
            if(map.containsKey(target))
            {
                //
                res = Math.max(res,i - map.get(target));
            }
            //如果此时map里面不包含当前正在循环的前缀和的值，将其和下标加入
            if(!map.containsKey(prefixNum[i]))
            {
                map.put(prefixNum[i],i);
            }
        }
        return res;
    }
```

