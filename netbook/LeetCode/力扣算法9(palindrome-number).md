# palindrome-number

## 一.题目

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

**示例 1:**

```
输入: 121
输出: true
```

https://leetcode-cn.com/problems/palindrome-number/

## 二.解题

先判断其是否为负数，再判断其是否为一位数，之后循环取模再将得到的取模结构乘10相加，则得到顺序倒转的结果。

## 三.代码

```java
 public static boolean isPalindrome(int x) {
        if (x<0){
            return false;
        }
        if (x<10){
            return true;
        }
        int temp=0;
        int sum=0;
        int x2=x;
        while (x!=0){
            temp=x%10;
            x/=10;
            sum=sum*10+temp;
        }
        if (x2==sum){
            return true;
        }
        return false;
    }
```

执行用时：**9 ms**

内存消耗：**39.4 MB**

## 四.分析

不再赘述。

## 五.优化

本题还可以再优化，可以考虑只反转 \text{int}int 数字的一半。如果该数字是回文，其后半部分反转后应该与原始数字的前半部分相同。

如何判断什么时候反转一半了呢？

当长度为奇数时则有，反转的数=剩下的数/10

当长度为偶数时则有，反转的数=剩下的数

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }
        int revertedNumber = 0;
        while (x > revertedNumber) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }
        return x == revertedNumber || x == revertedNumber / 10;
    }
}

```

