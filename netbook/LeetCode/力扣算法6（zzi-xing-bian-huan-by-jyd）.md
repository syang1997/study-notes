# zzi-xing-bian-huan-by-jyd

## 一.题目

https://leetcode-cn.com/problems/zigzag-conversion/solution/zzi-xing-bian-huan-by-jyd/

将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 "LEETCODEISHIRING" 行数为 3 时，排列如下：

L   C   I   R
		E T O E S I I G
		E   D   H   N
之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如："LCIRETOESIIGEDHN"。

## 二.解题

z字排列在从上到下的填充时到达两端时要进行递增递减的交换，所以本题将采用生成行数的list用于存放没一行的字符，然后遍历目标字符串在存放时到达第0个或者第n个时进行tigger的变化

### 三.代码

```java
public static String convert(String s, int numRows) {
        if (numRows==1){
            return s;
        }
            ArrayList<String> test[] = new ArrayList[numRows];
        for (int i = 0; i <numRows ; i++) {
            test[i]=new ArrayList<>();
        }
            int trigger=1;
            int num=0;
            int len=s.length();
        for (int i = 0; i < len; i++) {
            test[num].add(String.valueOf(s.charAt(i)));
            if (num==0){
                trigger=1;
            }else if (num==numRows-1){
                trigger=-1;
            }
            num+=trigger;
        }
        StringBuilder result=new StringBuilder();
        for (int i = 0; i <numRows ; i++) {
            for (String temp:test[i]) {
                result.append(temp);
            }
        }
        return result.toString();
    }
```

执行用时：**9 ms**

内存消耗：**40.2 MB**

## 四.分析

**复杂度分析：**

- 时间复杂度 O(N)*O*(*N*) ：遍历一遍字符串 `s`；
- 空间复杂度 O(N)*O*(*N*) ：各行字符串共占用 O(N)*O*(*N*) 额外空间。

## 五.优化

判断流程优化