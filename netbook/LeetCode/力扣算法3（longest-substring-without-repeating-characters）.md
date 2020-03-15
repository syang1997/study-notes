# longest-substring-without-repeating-characters

## 一.题目

https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

**示例 1:**

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

## 二.解题

使用了滑动窗口来解决问题，容器是hashmap；

```java
public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> map = new HashMap<>(s.length());
        int sum=0;
        int max=0;
        for (int i=0;i<s.length();i++){
            Integer exist = map.get(s.charAt(i));
            if (exist != null) {
                if(max<sum){
                    max=sum;
                    sum=0;
                }
                map.clear();
                i=exist;
            }else {
                map.put(s.charAt(i) ,i);
                sum=map.size();
            }
        }
        if(max<sum){
            max=sum;
            sum=0;
        }
        return max;
    }
```

## 三.结果

执行结果：

通过

显示详情

执行用时 :1136 ms, 在所有 Java 提交中击败了5.01%的用户

内存消耗 :42.6 MB, 在所有 Java 提交中击败了5.03%的用户

## 四.分析

由于使用了hash所以查询相同字符的时间复杂度为O(1),当最坏情况下窗口将在每个字符滑动，这时的复杂度为O(2n),最优势为*O*(*n*)，空间复杂度为O(min(m, n))。

## 五.优化

思路不变但是判断流程可以简化，

```java
 public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        Set<Character> set = new HashSet<>();
        int ans = 0, i = 0, j = 0;
        while (i < n && j < n) {
            // try to extend the range [i, j]
            if (!set.contains(s.charAt(j))){
                set.add(s.charAt(j++));
                ans = Math.max(ans, j - i);
            }
            else {
                set.remove(s.charAt(i++));
            }
        }
        return ans;
    }

```

