title: 112. Path Sum
date: 2016-09-09 15:02:26
tags: [leetcode,Tree，Deep-first Search]
categories: [leetcode]
---
# 题目描述
Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum.

For example:
Given the below binary tree and <code>sum = 22</code>,

<pre>              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
</pre>
return true, as there exist a root-to-leaf path <code>5->4->11->2</code> which sum is 22.
<!-- more -->
# 解题
## 思路
本题主要考察递归的用法，
## 代码
```java
**
 * Created with IntelliJ IDEA
 * Date: 2016/3/4
 * Time: 21:00
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */
public class PathSum_112 {
    public static void main(String[] args) {

    }

    /**
     *
     * @param root 根节点
     * @param sum 以root 为根的树的某一条路径的总和
     * @return 这棵树上是否存在这样一条路径
     */
    public boolean hasPathSum(TreeNode root, int sum) {

        if (root != null && root.left == null && root.right == null && sum - root.val == 0) {
            //如果当前节点是叶子节点，并且sum - 当前节点的值 =0 说明刚好找到了一条路径
            return true;
        } else if (root != null && root.left == null && root.right == null && sum - root.val != 0) {
            //如果当前节点是叶子节点，并且sum-root.val不为0 ，那么本条路径不满足sum条件
            return false;
        } else if (root != null){
            //如果当前节点不是叶子节点，继续往下递归查找，或的形式
            return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
        }else{
            //root等于null
            return false;
        }
    }
    public class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }
}
```
