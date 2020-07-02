# two-sum

## 一.题目

https://leetcode-cn.com/problems/two-sum/

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
		所以返回 [0, 1]

## 二.解题

第一反应想到的是暴力匹配，将target-第n个数与之后的数进行比较，由此遍历；

``` java
public int[] twoSum(int[] nums, int target) {
        for(int i=0 ;i<nums.length;i++){
            //用目标数区减第一个数然后向后依次比较
            int temp=target-nums[i];
            for (int j=i+1;j<nums.length;j++){
                if (temp==nums[j]){
                    int[] to={i,j};
                    return to;
                }
            }
        }
        return null;
    }
```

## 三.结果

![image-20200314164812636](image-20200314164812636.png)

| **29 / 29** 个通过测试用例              | 状态：通过 |
| --------------------------------------- | ---------- |
| 执行用时：**2 ms**内存消耗：**42.1 MB** |            |

## 四.分析

时间复杂度上是两次n循环为O(n2),空间复杂度为O(1);

经过查阅后可以同Hash来减少内层循环的时间复杂度，因为在比较遍历相等时是通过循环导致了内层的时间复杂度是O(n)，而hash查找的时间复杂度是O(1);

## 五.优化

```java
private  int[] hashSolution(int[] nums, int target) {
        if (nums == null || nums.length <= 1) {
            return null;
        }
        Map<Integer, Integer> map = new HashMap<>(nums.length);
        for (int i = 0; i < nums.length; i++) {
            int v = target - nums[i];
            Integer exist = map.get(v);
            if (exist != null) {
                return new int[] { exist, i };
            }
            map.put(nums[i], i);
        }

        return null;
    }
```

优化之后的时间复杂度是O(n) ,空间复杂度为O(n);