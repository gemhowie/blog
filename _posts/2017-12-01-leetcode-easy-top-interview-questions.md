---
layout: post
title: LeetCode Easy Top Interview Questions
categories: LeetCode
description: LeetCode Easy 难度，面试常考题
keywords: LeetCode, Python
---

LeetCode Easy 难度，面试常考题。

网页链接

[https://leetcode.com/problemset/all/?difficulty=Easy&listId=wpwgkgt](https://leetcode.com/problemset/all/?difficulty=Easy&listId=wpwgkgt)

所有solution均在Python3下通过测试。

## 1. Two Sum

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

Example:

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

给定一个整形数组，返回其中两个元素的下标，使得这两个元素相加等于指定的目标值。

### 思路

通过迭代`enumerate`对象，用`target - v`得到另一个要找的数，然后在剩下的元素中匹配，如果存在，则返回两个数的下标。

第二个数的下标因为是在切片后的数组中匹配，得到结果后要加上`i + 1`。

```python
class Solution:
    def twoSum(self, nums, target):
    """
    :type nums: List[int]
    :type target: int
    :rtype: List[int]
    """
    for i, v in enumerate(nums):
        second = target - v
        if second in nums[i+1:]:
            return [i, nums[i+1:].index(second) + i + 1]
```

## 7. Reverse Integer

Given a 32-bit signed integer, reverse digits of an integer.

Example 1:

```
Input: 123
Output:  321
```

Example 2:

```
Input: -123
Output: -321
```

Example 3:

```
Input: 120
Output: 21
```

### Note:

Assume we are dealing with an environment which could only hold integers within the 32-bit signed integer range. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

### 思路

反转32位有符号整数，溢出时返回`0`，通常的想法是转成`str`，然后用倒数切片，最后判断正负号和是否溢出。

```python
class Solution(object):
    def reverse(self, x):
        """
        :type x: int
        :rtype: int
        """
        y = int(str(abs(x))[::-1])
        return ((x > 0) - (x < 0)) * y * (y < 2**31)

        """
        python3 移除了cmp函数，在 python2 中可以这样写
        return cmp(x, 0) * y * (y < 2**31)
        """

# -2147483648 ~ 2147483647
# 7463847412
```

### 改进

溢出的判断不精确，还有如果不借助额外的空间，应该怎么解？

## 13. Roman to Integer

Given a roman numeral, convert it to an integer.

Input is guaranteed to be within the range from 1 to 3999.

给定一个罗马数字，将它转换成一个整数。输入值范围确保只从`1`到`3999`。

### 思路

先定义一个字典，键为罗马数字的字母，值为字母代表的值。

罗马数字有个特点，只有`1`，`5`，`10`，`50`，`100`...这样的值。

`2`和`3`就用基础的`1`来叠加，`II`代表`2`，`III`代表`3`。

`4`则有点特殊，用`5-1`表示，`1`放在`5`的左边，意为减去，所以`IV`代表`5`。

同样的，`6`，`7`，`8`用`5`和`1`来叠加，`VI`代表`6`，`VII`代表`7`，`VIII`代表`8`。

`9`和`4`类似，用`10-1`表示，`1`放在`10`的左边，所以`IX`代表`9`。

总结一下，就是，只有进位前一个数，比如`4`，`9`，`49`，`99`需要用到倒序减一的方式表示，其他的都是用最靠近的数加上基础数`(1，10，100...)`的个数来表示。

所以可以循环罗马数字，相加减的次数(循环的次数)应该是罗马数字的个数减1，但是我们会用到一个变量来存储累加结果，所以次数会加1。

我们只把罗马数字的前`len(s)-1`个字母放在循环里处理，最后一个字母通过`return`时的表达式给加上去，这样就可以处理只有一个罗马数字的情况。

```python
class Solution:
    def romanToInt(self, s):
        """
        :type s: str
        :rtype: int
        """
        roman = {'M': 1000, 'D': 500, 'C': 100, 'L': 50, 'X': 10, 'V': 5, 'I': 1}
        z = 0
        for i in range(0, len(s) - 1):
            if roman[s[i]] < roman[s[i+1]]
                z -= roman[s[i]]
            else:
                z += roman[s[i]]
        return z + roman[s[-1]]
```

## 14. Longest Common Prefix

Write a function to find the longest common prefix string amongst an array of strings.

写一个函数从一个字符串数组中查找最长的共同前缀字符串。

### 思路

Python的`os`模块已经实现这个功能，`os.path.commonprefix(strs)`，但通常情况下面试官不希望看到这个答案。:smile:

第二个方法是使用`zip`函数。

在Python2中，`zip`函数接收多个可迭代对象，分别把多个迭代对象的第1个，第2个...第n个元素(n取决于可迭代对象中长度最短的那一个)组成`tuple`，然后返回一个包含这些`tuple`的`list`。

在Python3中，`zip`函数的返回值变成了`zip`对象，想得到Python2的结果，可以通过`list()`来转换`zip`对象。

具体思路是，用`*strs`将数组中的字符串作为多个可迭代对象参数传给`zip`，然后用`enumerate`获取`index`和对应的`tuple`。

如果有共同前缀，那么`tuple`中的元素应该都是相同的，用`set`转换可以去重，然后`len`判断元素个数。

如果从某个`index`对应的`tuple`开始，去重后元素个数大于1，说明字符已经不相同，只需要截取当前`index`前的部分，即为最长共同前缀。

`for..else..`的写法有个陷阱，当`for`循环中不存在`return`或`break`时，循环结束后会继续执行`else`部分的内容。

下面这个例子，`for`中的`if`如果都不存在满足的条件，`return`就不会执行，循环结束后执行`else`内容。这意味着每个`tuple`中都只包含同一个元素，`else`中返回`min(strs)`或者`min(strs, key=len)`即为最长共同前缀。

`return strs[0][:i]`需要解释一下，当输入是`['z', 'abc']`时，经过`zip`函数处理后的`tuple`是`('z', 'a')`，元素个数大于1。

也就是`index`为`0`时就不存在相同的字符了，`strs[0][:i]`即`strs[0][:0]`，结果是`''`，所以不必担心`else`中使用`min(strs)`会根据`ASCII`返回`abc`这种结果，因为这种情况下根本不会走`else`分支。只是像这样依赖于条件控制，有时会使人产生困惑。

```python
class Solution(object):
    def longestCommonPrefix(self, strs):
        """
        :type strs: List[str]
        :rtype: str
        """
        if not strs:
            return ""
        
        for i, letter_group in enumerate(zip(*strs)):
            if len(set(letter_group)) > 1:
                return strs[0][:i]
        else:
            return min(strs)
```

第三个方法，用`functools`的`reduce`函数。

## 20. Valid Parentheses

Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

给定一个只包含`'('，')'，'['，']'，'{'，'}'`的字符串，判断输入是否有效。

括号必须以正确的顺序结束。

### 思路

循环输入的字符串，碰到开始括号则入栈，碰到关闭括号则从写好的字典中获取对应的开始括号，和栈顶出栈的括号比较，如果不等，则输入无效，相等则循环直到栈为空。

```python
class Solution:
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        l = len(s)

        if l == 0:
            return True
        
        if l % 2 != 0:
            return False
        
        check = {')': '(', ']': '[', '}': '{'}
        stack = []
        for i in s:
            if i in ('(', '[', '{'):
                stack.append(i)
            else:
                if not stack or check[i] != stack.pop():
                    return False
        return len(stack) == 0
```

## 21. Merge Two Sorted Lists

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

Example:

```
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```

合并两个有序链表并作为一个新链表返回。新的链表应该通过拼接前两个链表的节点来生成。

### 思路

递归版本，函数的主要功能是比较两个链表，确保`l1`不为空节点，并且`l1`第一个节点中值最小，如果不满足，交换`l1`和`l2`，`l1`作为新链表的第一个节点，`l1.next`则重新指向通过递归调用函数本身，得到第二小的元素节点，直到某个递归深度时其中一个`list`为空，最后返回`l1`。

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        if l1 is None or l2 and l1.val > l2.val:
            l1, l2 = l2, l1
        if l1 and l2:
            l1.next = self.mergeTwoLists(l1.next, l2)
        return l1
```

循环版本

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution():
    def mergeTwoLists(self, l1, l2):
        """
        :type l1: ListNode
        :type l2: ListNode
        :rtype: ListNode
        """
        dummy = cur = ListNode(0)
        while l1 and l2:
            if l1.val > l2.val:
                l1, l2 = l2, l1
            cur.next = l1
            l1 = l1.next
            cur = cur.next
        cur.next = l1 or l2
        return dummy.next
```

## 26. Remove Duplicates from Sorted Array

Given a sorted array, remove the duplicates in-place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

Example:

```
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.
It doesn't matter what you leave beyond the new length.
```

给定一个有序的数组，通过原位操作移除重复元素，让每个元素只出现一次并返回新的长度。

不能分配额外的空间给另一个数组，你必须在O(1)的额外内存中通过原位操作修改输入的数组来完成它。

```
给定 nums = [1,1,2]，

你的函数应该返回长度为2，且nums最开始的两个元素分别为1和2。无论在新长度之后留下什么都没关系。
```

### 思路

因为需要`in-place`，即原位操作，而且只能用`O(1)`的额外内存，所以不能用数组这种会随着规模变大而变大的临时变量。

用一个变量`i`来表示新长度，用`for n in nums`遍历这个`nums`，比较后一个元素与前一个元素，因为`nums`是有序的，如果后一个元素大于前一个，则通过`nums[i] = n`覆盖原来的值，并且`i`自增，如果比较结果相同，`i`不变，继续比较下一个元素。直到遍历完`nums`，`i`的值即为不重复元素的个数。

```python
class Solution:
    def removeDuplicates(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        i = 0
        for n in nums:
            if i == 0 or n > nums[i-1]:
                nums[i] = n
                i += 1
        return i
```
