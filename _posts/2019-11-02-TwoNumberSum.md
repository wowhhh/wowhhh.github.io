---
layout: post
title:  "02 两数相加"
tags:   链表 LinkedList 面试 LeetCode
date:   2019-11-02 
categories: [Algorithm]

---

# 目录

- [题目描述](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#题目描述)
- [我的解答与思路](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#我的解答与思路)

- [解析](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#解析)
  - [思路](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#思路)
  - [伪代码](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#伪代码)
  - [代码重写](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#代码重写)
  - [正解](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#正解)
  - [知识点](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#知识点)
  - [优化](https://github.com/wowhhh/LeetCode/blob/master/01_链表/2.两数相加.md#优化)
## 题目描述
- 给出两个非空的链表用来表示两个非负的整数。其中，它们各自的位数是按照逆序的方式存储的，并且它们的每个节点只能存储一位数字。
如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
您可以假设除了数字 0 之外，这两个数都不会以 0 开头。
```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```
## 我的解答与思路
- 蠢的一批的解法，刚开始的解决方案是按照 一位对一位计算，写出来之后，发现好多情况都不行，GG，就对每种情况一个一个做.....接着就出现了这么多Bug。



```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l3 = new ListNode(0);
        ListNode temp =l3;
        int flag = 0;//标志一下当前进位不
        while(l1 !=null && l2 !=null) //循环条件
        {
            l3.val = l1.val + l2.val + flag;
            flag = 0;
            if(l3.val >=10 && l1.next!=null && l2.next!=null)
            {
                flag = 1;
                l3.val = l3.val -10;
            }

            //创建l3的下一个结点
            System.out.println(l3.val);
            if(l3.val>=10 && (l1.next==null || l2.next ==null))//如果最后两位的和大于10
            {
                l3.next = new ListNode(0);
                l3.next.val = l3.val/10;
                l3.val = l3.val%10;
                l1 = l1.next;
                l2 = l2.next;
                l3 = l3.next;
                flag = 1;
            }
            else
            {
            if((l1.next==null) &&(l2.next==null))
            {
                l1 = l1.next;
                l2 = l2.next;
            }
            else
            {
            l3.next = new ListNode(0);
            l3 = l3.next;
            l1 = l1.next;
            l2 = l2.next;
            }
            }
        }
        if(l1!=null )
        {

            l3.val = l1.val+flag;
        }
        if(l2!=null)
        {
            l3.val = l2.val+flag;
        }
        if(l3.val >=10)
        {
            l3.next = new ListNode(0);
            l3.next.val = l3.val/10;
            l3.val = l3.val%10;
        }
        return temp;
    }
}
```
- 上述算法很糟糕并且没有实现想要的功能

 

## 解析
### 思路
- 设置进位(想到了)
### 伪代码
- 将当前结点初始化为返回列表的**哑结点**
- 将进位carry初始化为0
- 将p和q结点分别初始化为l1和l2的头结点
- 遍历链表l1和l2直至他们的尾端
  - 将x的值设置为p结点的值。**如果p已经到达了l1链表的尾端，将x的值设置为0。**
  - 将y的值设置为q结点的值。**如果q结点已经到达了l2链表的尾端，将y的值设置为0。**
  - 设置 *sum = x + y + carry*
  - 更新进位的值：*carry = sum/10*
  - 创建一个数值为(*sum%10*)的新结点，并将当前结点的下一个结点设置为此节点，再讲当前结点前进到下一个结点，也就是前进此时新建的结点
  - p,q前进一个结点
- 检查carry = 1是否成立，如果成立，则向返回列表追加一个含有数字1的结点
- 返回哑结点的下一个结点
### 代码重写
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode l3 = new ListNode(0);
        ListNode temp = new ListNode(0);
        temp = l3;
        //先判断两个谁长
        int flag = 0;
        ListNode p = l1;
        ListNode q = l2;
        // 错 while(p.next!=null && q.next !=null)
        {
          /*
            int x = p.val;
            int y = q.val;
            // 错 if(p.next==null) {x = 0;}
            // 错 if(q.next==null) {y =0;}
            错
          */
            int sum = x + y + flag;
            flag = sum/10;
            ListNode value = new ListNode(sum%10);
            l3.next = value;
            l3 = l3.next;
            if(p != null) p = p.next;
            if(q != null) q = q.next;
        }
        if(flag==1)
        {
            ListNode add = new ListNode(flag);
            l3.next = add;
        }
        return temp.next;
    }
}
```
- ![](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/markdown-img-paste-20191101220825376.png)
- 错误点：
  - 循环条件
  - 指针的操作，判空、取值之类的
  - 定义的结点太多，内存占用太大
### 正解
```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2 , temp = dummyHead;
    int carry = 0;//默认进位为0
    while(p!=null || q!=null)//两者一方不为空就继续循环
        //但是要在循环代码里面处理好一方为空的情况
    {
        //取出 p,q的值
        int x = (p != null)?p.val:0;
        int y = (q != null)?q.val:0;
        //取和
        int sum = x + y +carry;
        //设置下一个进位
        carry = sum/10;
        //创建新结点，保存两数之和
        temp.next = new ListNode(sum%10);
        temp = temp.next;
        //循环q,p
        if(q!=null) q=q.next;
        if(p!=null) p=p.next;
    }
    //处理carry!=0的情况
    if(carry!=0)
    {
        temp.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```
### 知识点
- 哑结点的使用
- 循环条件
  - 一方到头置为0
- 进位的使用
### 代优化

![](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/markdown-img-paste-2019110218162096.png)
