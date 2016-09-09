title: 102. Binary Tree Level Order Traversal
date: 2016-09-09 09:26:34
tags: [leetcode,tree,Breadth-first Search]
categories: [leetcode]
---
# 题目描述
Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).

For example:
Given binary tree [3,9,20,null,null,15,7],
<pre>
3
/ \
9  20
 /  \
15   7
</pre>

return its level order traversal as:
<pre>
[
  [3],
  [9,20],
  [15,7]
]
</pre>

<!-- more -->
# 解题
## 思路
使用队列，每次将一层加入到当前队列。
## 代码
全部代码
```java
import java.util.*;

/**
 * Created with IntelliJ IDEA
 * Date: 2016/3/4
 * Time: 19:50
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */
public class BinaryTreeLevelOrderTraversal_102 {
    public static void main(String[] args) {
        TreeNode treeNode = new TreeNode(1);
        levelOrder(treeNode);
    }

    public static List<List<Integer>> levelOrder(TreeNode root) {
        //null的处理
        if (root == null) {
            return new ArrayList<List<Integer>>();
        }
        //返回值
        List<List<Integer>> lists = new ArrayList<List<Integer>>();
        //层次遍历所需要的队列
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        //root 不为空，将root加入队列中
        queue.add(root);
        //如果队列不为空
        while (!queue.isEmpty()) {
            //保存当前对别中的所有节点
            ArrayList<TreeNode> treeNodes = new ArrayList<TreeNode>();
            //保存当前队列中所有节点的值
            ArrayList<Integer> vals = new ArrayList<Integer>();
            //取出当前队列中的所有节点
            Iterator<TreeNode> iterator = queue.iterator();
            while (iterator.hasNext()) {
                TreeNode temp = iterator.next();
                treeNodes.add(temp);
                vals.add(temp.val);
            }
            //将当前队列中的所有值最为一个list加入到 返回值列表
            lists.add(vals);
            //当前队列清空
            queue.clear();
            //将treeNodes中保存的当前队列中的所有值的左节点和右节点 进行判断。
            for (TreeNode treeNode : treeNodes) {
                TreeNode temp = treeNode;
                if (temp.left != null) {
                    queue.add(temp.left);
                }
                if (temp.right != null) {
                    queue.add(temp.right);
                }
            }
        }
        return lists;
    }

    public static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }
}
```
