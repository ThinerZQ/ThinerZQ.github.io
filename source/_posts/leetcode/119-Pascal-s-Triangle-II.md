title: 119.Pascal's Triangle II
date: 2016-09-09 14:43:26
tags: [leetcode,array]
categories: [leetcode]
---
# 题目描述
Given an index k, return the k<sup>th</sup> row of the Pascal's triangle.

For example, given k = 3,

Return

<pre>[1,3,3,1].
</pre>
本题k=0,返回[1]
<!-- more -->
# 解题
## 思路
每一行除了第0个元素和最后一个元素除外，current[j] = lastRow[j-1]+lastRow[j];
## 代码
```java
public List<Integer> getRow(int rowIndex) {
        List<Integer> pre = new ArrayList<Integer>();
        pre.add(0,1);

        for (int i = 0; i<rowIndex; i++) {
            List<Integer> tempLists =new ArrayList<Integer>();
            tempLists.add(0,1);
            for (int j = 1; j <=i;j++) {
                tempLists.add(j,pre.get(j-1)+pre.get(j));
            }
            tempLists.add(1);
            pre = tempLists;
        }
        return pre;
    }
```
