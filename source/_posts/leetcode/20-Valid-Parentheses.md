title: 20. Valid Parentheses
tags: [Stack,String]
categories: [leetcode]
date: 2016-09-09 16:00:46
---
# 题目描述
Given a string containing just the characters <code>'(', ')', '{', '}', '[' and ']'</code>, determine if the input string is valid.

The brackets must close in the correct order, <code>"()" and "()[]{}" are all valid but "(]" and "([)]"</code> are not.

<!-- more -->
# 解题
## 思路
字符串里面只包含括号，使用出栈入栈判断揪心了。
## 代码
```java
import java.util.Stack;

/**
 * Created with IntelliJ IDEA
 * Date: 2016/3/18
 * Time: 22:32
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */
public class ValidParentheses_20 {
    public static void main(String[] args) {

    }

    public boolean isValid(String s) {
        //括号问题当然要使用栈啦
        Stack<Character> stack = new Stack<Character>();
        char[] characters = s.toCharArray();
        //对每一个字符
        for (char c : characters) {
            if (c == '(' || c == '{' || c == '[') {
                //如果 c 是括号类‘左’字符入栈
                stack.push(c);
            } else if (c == ')' && !stack.isEmpty() && stack.peek() == '(') {
                //如果 c 是括号类 ‘右’ 字符，并且栈顶是 c对应的 ‘左’字符，匹配上了，pop
                stack.pop();
            } else if (c == '}' && !stack.isEmpty() && stack.peek() =='{') {
                stack.pop();
            } else if (c == ']' && !stack.isEmpty() && stack.peek()=='[') {
                stack.pop();
            }else{
                return false;
            }
        }
        return stack.isEmpty();
    }
}

```
