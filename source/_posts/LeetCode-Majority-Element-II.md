---
title: LeetCode(229) Majority Element II
tags:
  - 算法
  - LeetCode
  - Algorithm
date: 2020-09-22 22:26:41
---


## [Question](https://leetcode.com/problems/majority-element-ii/)
Given an integer array of size n, find all elements that appear more than ⌊ n/3 ⌋ times.

Note: The algorithm should run in linear time and in O(1) space.

<b>Example 1:</b>
```
Input: [3,2,3]
Output: [3]
```
<b>Example 2:</b>
```
Input: [1,1,1,3,3,2,2,2]
Output: [1,2]
```

## Solution
The solution for this problem relies on one observation, which is that, for any array, **there will have at most 2 elements that occurs more than ⌊ n/3 ⌋ times.** After knowing this, we shall be able to achieve the O(1) space as we only rely on 4 variables here.

Fisrtly, we require 2 sets of (element, count) as we would like to find the top 2 majority element.

Secondly, it is known that, if an element is a majority element, the count CANNOT be 0. 
For example, if we have an array of size 10 `[1, 1, 1, 1, 2, 3, 4, 5, 6, 7]`, to be a majority element, it must occurs 4 times. We will have, for first pair of (element, count), we can have (1, 4), and second element it will be used to allocate another 1/3 of elements. In this case, the element 1 will be always having more than 0 counts.

If there are 2 majority elements, the logic still apply. We will have 2 paris to allocate the count, only left with at most 3 elements to be reduced. Therefore, both counts won't be 0, which implies that the candidates will not be replaced.


```Go
func majorityElement(nums []int) []int {
    
     var results []int;
    var count1 int = 0;
    var count2 int = 0;
    
    if len(nums) == 0 || nums == nil {
        return results
    }
    
    var candidate1 int = nums[0];
    var candidate2 int = nums[0];
  
    
    for i:=0; i < len(nums); i++ {
        
        if candidate1 == nums[i] {
            count1++
        } else if candidate2 == nums[i] {
            count2++
        } else if count1 == 0 {
            candidate1 = nums[i]
            count1++
        } else if count2 == 0 {
            candidate2 = nums[i]
            count2++
        } else {
            count1--
            count2--
        }
        
    }
    
    count1 = 0
    count2 = 0
    
    for i:=0; i < len(nums); i++ {
        if nums[i] == candidate1 {
            count1++
        } else if nums[i] == candidate2 {
            count2++
        }
    }
    
    limit := len(nums)/3
    
    if limit != 0 {
        limit += 1
    }
    
    if count1 >= limit {
        results = append(results, candidate1)
    }
    
    if count2 >= limit && candidate1 != candidate2 {
        results = append(results, candidate2)
    }
    
    return results
}
```