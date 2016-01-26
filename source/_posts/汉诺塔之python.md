title: 汉诺塔之python
date: 2016-01-26 14:02:04
tags:
 -汉诺塔
 -python
categories: python
---

# 解法
# 代码
```python
# -*- coding: cp936 -*-  
def hannoi(n,x,y,z):  
    if n==1:  
        print(x," ---->",z)  
    else:  
        hannoi(n-1,x,z,y)#将前n-1个盘子从x移动到y上  
        print(x,"-->",z)#将最底下的一个胖子从x移动到z上  
        hannoi(n-1,y,x,z)#将y上的n-1个盘子移动到z上  
n=int(input("请输入汉诺塔的层数："))  
hannoi(n,"x","y","z")  
```
# 结果
![result](/images/hannui_result.png)


