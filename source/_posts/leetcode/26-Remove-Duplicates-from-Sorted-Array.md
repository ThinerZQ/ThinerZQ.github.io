title: 26.Remove Duplicates from Sorted Array
date: 2016-09-09 10:01:04
tags: [leetcode,two pointers]
categories: [leetcode]
---

# 题目描述
Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

For example,
Given input array nums =<code> [1,1,2]</code>,

Your function should return length = <code>2</code>, with the first two elements of nums being<code> 1</code> and <code>2</code> respectively. It doesn't matter what you leave beyond the new length.


<!-- more -->
# 解题
## 思路
判断当前值和前一个值是否相等，相等就记录下有多少个相等的，不相等就将当前值往前赋值。
## 代码
```java

/**
 * Created by root on 16-3-3.
 */
public class RemoveDuplicatesFromSortedArray_26 {
    public static void main(String[] args) {

        int nums[] = {1,1,2,2,3,4,4,3};
        System.out.println(new RemoveDuplicatesFromSortedArray_26().removeDuplicates(nums));
        for (int i = 0; i < nums.length; i++) {
            System.out.println(nums[i]);
        }
    }

    public int removeDuplicates(int[] nums) {
        int count = 0;
        // 一个for 循环
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] == nums[i - 1]) {
                //当前值 和 前一个值是否相等，相等 count+1
                count++;
            } else {
                // count 永远是 小于 1 的
                //经过上面的判断，
                // count ==0    《--》 前面没有相等的
                // count ==1     《--》  前面有一个相等的，将上一个值赋值为当前值
                // count ==2     《--》  前面有两个相等的，将上两个位置的值赋值为当前值
                // ...
                nums[i - count] = nums[i];
            }
        }
        //返回新的数组长度
        return nums.length - count;
    }
}
```
