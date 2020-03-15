# add-two-numbers

## 一.题目

https://leetcode-cn.com/problems/add-two-numbers/submissions/

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807

## 二.解题

想到的是数相加然后判断取其进位，然后再将每一位数据插入到链表中；

```java
public  ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int temp=0;
        ListNode root=null;
        ListNode t=root;
        while (true){
            temp=l1.val+l2.val+temp;
            if (temp>9){
                ListNode l3=new ListNode(temp%10);
                if (root==null){
                    root=l3;
                    t=root;
                }else {
                    t.next=l3;
                    t=t.next;
                }
                temp=1;
            }else {
                ListNode l3=new ListNode(temp);
                temp=0;
                if (root==null){
                    root=l3;
                    t=root;
                }else {
                    t.next=l3;
                    t=t.next;
                }
            }


            if (l1.next==null&&l2.next==null){
                if (temp>0){
                    t.next=new ListNode(1);
                    t=t.next;
                }
                return root;
            }
            if (l1.next!=null){
                l1=l1.next;
            }else {
                l1.val=0;
            }
            if (l2.next!=null){
                l2=l2.next;
            }else {
                l2.val=0;
            }
        }
    }
```

## 三.结果

执行结果：

通过

显示详情

执行用时 :3 ms, 在所有 Java 提交中击败了39.12%的用户

内存消耗 :41.4 MB, 在所有 Java 提交中击败了92.66%的用户

## 四.分析

操作都是链表的遍历操作，有两个链表和一个结果链表，但是由于判断只会进行一次，所以时间复杂度为O(max(m, n)),空间复杂度为*O*(max(*m*,*n*));

## 五.优化

思路不变代码的判断过程可以优化

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}

```

