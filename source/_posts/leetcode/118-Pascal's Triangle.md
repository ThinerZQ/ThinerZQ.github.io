title: 102 Pascal'a Triangle
date: 2016-09-08 21:08:06
tags: [leetcode,array]
categories: [leetcode]
---
# 题目描述
given numRows, generate the first numRows of Pascal's triangle.

For example, given numRows = 5

Return

<pre>[
     [1],
    [1,1],
   [1,2,1],
  [1,3,3,1],
 [1,4,6,4,1]
]
</pre>
<!-- more -->
# 解题
## 思路
每一行除了第0个元素和最后一个元素除外，current[j] = lastRow[j-1]+lastRow[j];
## 代码
```java
public static List<List<Integer>> generate(int numRows) {
        //防御式编程
        if (numRows == 0) {
            return new ArrayList<List<Integer>>();
        }
        //最终返回结果
        List<List<Integer>> lists = new ArrayList<List<Integer>>();
        //加入第一行元素
        ArrayList<Integer> first = new ArrayList<Integer>();
        first.add(0, 1);
        lists.add(0, first);
        //对接下来的每一行，取出上一行
        for (int i = 1; i < numRows; i++) {
            //templist 是arraylist，可以保持加入的顺序
            List<Integer> tempLists = new ArrayList<Integer>();
            //取出上一行
            List<Integer> preList = lists.get(i - 1);
            //在templist 上插入第0个元素：1，
            tempLists.add(0, 1);
            // int len = (i)/2+1;
            //本行的第j个元素 = 上一行的第j-1个元素 + 上一行的第j个元素 ， 1<=j<i
            for (int j = 1; j < i; j++) {
                tempLists.add(j, preList.get(j - 1) + preList.get(j));
            }
            //在templist 中Haru最后一个元素：1
            tempLists.add(1);
            lists.add(i, tempLists);
        }
        return lists;
    }
```
