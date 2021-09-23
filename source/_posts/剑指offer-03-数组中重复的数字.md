---
title: 剑指offer-03-数组中重复的数字
date: 2021-06-17 06:19:21
categories: 
- Algorithm
tags:
- leetcode
- 剑指offer
- hash-table
---

## 题目描述

题目链接:[剑指 Offer 03. 数组中重复的数字 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

找出数组中重复的数字。

在一个长度为` n` 的数组 `nums `里的所有数字都在 `0`～`n-1` 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。



示例1：

```
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```



需要考虑到时间复杂度和空间复杂度的要求

如果要求时间复杂度，可以考虑用hash table.(时间复杂度 `O(N)`   空间复杂度 `O(N)`)

如果要求空间复杂度，可以先排序，再看前后有无重复数字.(时间复杂度 `O(Nlog(N))`   空间复杂度 `O(1)`)

如果都有要求，考虑原地置换法 .(时间复杂度 `O(N)`   空间复杂度 `O(1)`)