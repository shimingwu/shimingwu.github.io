---
title: LeetCode Largest Number
tags:
  - 算法
  - LeetCode
  - Algorithm
date: 2020-09-26 12:29:08
---

## [Question](https://leetcode.com/problems/largest-number/submissions/)
Given a list of non negative integers, arrange them such that they form the largest number.

<b>Example 1:</b>
```
Input: [10,2]
Output: "210"
```
<b>Example 2:</b>
```
Input: [3,30,34,5,9]
Output: "9534330"
```

## Solution 1
```Go
func largestNumber(nums []int) string {

    var numStr []string;
     for i:=0; i < len(nums); i++ {            
         numStr = append(numStr, strconv.Itoa(nums[i]))
    }
    
    var result []string;
    result = quickSort(numStr, 0, len(numStr)-1)
    
    var str string
    
    for i:=len(result)-1; i >=0; i--{
        str += result[i]
    }
    
    if str[0] == '0' {
        str = "0"
    }
    

    return str
}


func quickSort(nums []string, left int, right int) []string {

    if left > right {
        return nums
    }
    
    i := left
    j := right
    
    pivot := nums[left]
    for i != j {
        
        for compare(nums[j], pivot) && i < j {
            j--;
        }
        
        for compare(pivot, nums[i]) && i < j {
            i++;
        }
        
        if i < j {
            temp := nums[i];
            nums[i] = nums[j]
            nums[j] = temp
        }
        
    }
    
    nums[left] = nums[i];
    nums[i] = pivot;
    
    quickSort(nums, left, i-1);
    quickSort(nums, i+1, right);
    
    return nums
}


func compare(num1 string, num2 string) bool {

    if num1 == num2 {
        return true
    }
    
    len1 := len(num1)
    len2 := len(num2)
    
    maxLen := len1*2
    if len2 > len1 {
        maxLen = len2*2
    }
    
    for i:=0 ; i < maxLen; i++ {
        num1Digit := num1[i % len1]
        num2Digit := num2[i % len2]
        if num1Digit > num2Digit {
            return true
        } else if num1Digit < num2Digit {
            return false
        }
    }

    
    return true
}
```


## Solution 2
```Go
var pow10 = [10]int {1, 10, 100, 1000, 10000, 100000, 1000000, 10000000, 100000000, 1000000000}
 
func largestNumber(nums []int) string {
    
    var result []int;
    result = quickSort(nums, 0, len(nums)-1)
    
    var str string
    
    for i:=len(result)-1; i >=0; i--{
        str += strconv.Itoa(result[i])
    }
    
    if str[0] == '0' {
        str = "0"
    }
    

    return str
}


func quickSort(nums []int, left int, right int) []int {

    if left > right {
        return nums
    }
    
    i := left
    j := right
    
    pivot := nums[left]
    for i != j {
        
        for compare(nums[j], pivot) && i < j {
            j--;
        }
        
        for compare(pivot, nums[i]) && i < j {
            i++;
        }
        
        if i < j {
            temp := nums[i];
            nums[i] = nums[j]
            nums[j] = temp
        }
        
    }
    
    nums[left] = nums[i];
    nums[i] = pivot;
    
    quickSort(nums, left, i-1);
    quickSort(nums, i+1, right);
    
    return nums
}


func compare(num1 int, num2 int) bool {

    if num1 == num2 {
        return true
    }
    
    len1 := len(strconv.Itoa(num1))
    len2 := len(strconv.Itoa(num2))
    
    maxLen := len1*2
    if len2 > len1 {
        maxLen = len2*2
    }
    
    for i:=0 ; i < maxLen; i++ {
        
        num1Digit := num1 / pow10[len1 - (i % len1 + 1)] % 10
        num2Digit := num2 / pow10[len2 - (i % len2 + 1)] % 10
        
        if num1Digit > num2Digit {
            return true
        } else if num1Digit < num2Digit {
            return false
        }
    }

    
    return true
}
```