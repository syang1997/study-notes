# reverse-integer

## 一.题目

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

**示例 1:**

```
输入: 123
输出: 321
```

https://leetcode-cn.com/problems/reverse-integer/

## 二.解题

只需要进行取模再除十的操作就可以取出每一位上的数字，再将其乘十相加即可

## 三.代码

```java
 public int reverse(int x) {
        int rev = 0;
        while (x != 0) {
            int pop = x % 10;
            x /= 10;
            if (rev > Integer.MAX_VALUE/10) return 0;
            if (rev < Integer.MIN_VALUE/10) return 0;
            rev = rev * 10 + pop;
        }
        return rev;
    }
```

执行用时：**1 ms**

内存消耗：**37.1 MB**

## 四.分析

因为32位的int范围-2147683648-2147683647，在倒转然后其益处的情况是要在位数要满的情况下，在此前提下末尾的一个数字就是未倒装时的最高位只能为1或者2，所以无需判断其>7和8的情况。

