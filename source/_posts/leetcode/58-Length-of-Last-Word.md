title: 58. Length of Last Word
tags: []
categories: []
date: 2016-09-09 16:14:49
---
# 题目描述
Given a string s consists of upper/lower-case alphabets and empty space characters <code>' '</code>, return the length of last word in the string.

If the last word does not exist, return 0.

Note: A word is defined as a character sequence consists of non-space characters only.

For example,
Given <code>s = "Hello World"</code>,
return <code>5</code>.

<!-- more -->
# 解题
## 思路
这道题是java太好写了吗？？
## 代码
```java

/**
 * Created with IntelliJ IDEA
 * Date: 2016/3/19
 * Time: 16:11
 * User: ThinerZQ
 * GitHub: <a>https://github.com/ThinerZQ</a>
 * Blog: <a>http://www.thinerzq.me</a>
 * Email: 601097836@qq.com
 */
public class LengthofLastWord_58 {

    public static void main(String[] args) {

    }

    public int lengthOfLastWord(String s) {

        if (s == null || s.trim().length() == 0) {
            return 0;
        } else {
            String[] strings = s.split(" ");
            return strings[strings.length - 1].length();
        }
    }
}

```
