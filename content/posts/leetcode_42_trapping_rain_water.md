---
title: [leetcode]  42. trapping rain water
date: 2024-04-26
tags:
- leetcode
categories:
- algorithm
description: "idea: solving problems by two pointers"
---

#  [Solution] 42. 接雨水

[力扣题目链接](https://leetcode.cn/problems/trapping-rain-water/)

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

示例 1：

![Alt text](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/22/rainwatertrap.png)

输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6

解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 

> 思路: 盛水的地块，一定是 min (左边最高的, 右边最高）&& （当前比左右两边都地），那如何找到左右最高的部分，就成为了关键解题思路。

`left` ,`right` 表示移动的指针，需要计算当前 位置能盛水的量。
`leftMax`, `rightMax` 表示当前左边，右边最高的位置，(`left` ,`right`) 一定是在 (`leftMax`, `rightMax`) 范围内。

每次移动时，小的往里面移动，大的不动，
如果移动之后，新小的比较之前小的大了，那么更新对应max标志位，这首也不需要计算水位。
如果移动之后，新小的比较之前小的小了，那么不需要更新max标志位，但需要计算当前坑位的水位，并加入 结果中

The approach involves identifying the land that can contain water, which is contingent upon the minimum height between the tallest barrier to the left and the tallest to the right, with the current position being lower than both. The crux of the solution is to determine the tallest sections on either side.
left and right are indices that represent the moving pointers, and it is necessary to calculate the volume of water that can be contained at the current location.
leftMax and rightMax denote the positions of the highest barriers currently on the left and right, respectively, with the stipulation that the indices (left, right) are always within the boundaries of (leftMax, rightMax).
During each iteration, the pointer associated with the smaller height is moved inward, while the one with the larger height stays in place.
If, after moving, the new smaller height is greater than the previous smaller height, then update the corresponding maximum height flag; in this case, there is no need to calculate the water level.
If, after moving, the new smaller height is less than the previous smaller height, then there is no need to update the maximum height flag. However, you must calculate the water volume for the current depression and include it in the result.

```
func trap(height []int) int {
	left := 0
    right := len(height) - 1
    leftMax := 0
    rightMax := len(height) - 1
    res := 0
    for left < right {
        if height[left] < height[right] {
            left++
            if height[left] > height[leftMax] {
                leftMax = left
            } else {
                res = res + min (height[leftMax], height[rightMax]) - height[left]
            }
        } else {
            right--   
            if height[right] > height[rightMax] {
                rightMax = right
            } else {
                res = res + min (height[leftMax] , height[rightMax]) - height[right]
            }
        }
    }
    return res
}

func min(x, y int) int {
    if x < y {
        return x
    } else {
        return y
    }
}
```