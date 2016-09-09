title: 19. Remove Nth Node From End of List
tags: [Linked List, Two Pointers]
categories: [leetcode]
date: 2016-09-09 15:32:22
---
# 题目描述
Given a linked list, remove the nth node from the end of list and return its head.

For example, Given linked list:
<pre>   
     <b>1-&gt;2-&gt;3-&gt;4-&gt;5</b>
</pre>

 and n=2, After removing the second node from the end , the linked list becomes
 <pre>1->2->3->5.
</pre>

Note:
Given n will always be valid.
Try to do this in one pass.

<!-- more -->
# 解题
## 思路
使用两个指针，第一个指针先移动n步，然后两个指针再同时移动直到第一个指针到达末尾，这时候第二个指针所在的位置就是到数第n位
## 代码
```java
/**
 * Created with IntelliJ IDEA
 * Date: 2016/3/19
 * Time: 15:50
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */
public class RemoveNthNodeFromEndofList_19 {

    public ListNode removeNthFromEnd(ListNode head, int n) {
        //null判断
        if (head == null) {
            return null;
        }
        //定义两个节点
        ListNode next = head;
        ListNode pre = head;
        //先计算出顺数第n个的位置
        for (int i = 0; i < n; i++) {
            next = next.next;
        }
        //如果顺数第n的位置为null, 标示到数第n个元素是head, 将head移除。
        if (next == null) {
            head = head.next;
            return head;
        }
        //顺数第n为不为null,并行移动向后pre 和next两个为止，直到next为空，表明：pre移到了倒数第n为的位置
        while (next.next != null) {
            next = next.next;
            pre = pre.next;
        }
        //位置交换，断开到数第n为的节点
        ListNode temp = pre.next.next;
        pre.next = temp;
        return head;

    }

    public class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }
}
```
