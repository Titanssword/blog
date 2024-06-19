---
title: sort algorithm 
date: 2024-04-26
tags:
- leetcode 
categories:
- algorithm
description: "The Top-K Problem involves finding the K largest or smallest elements in a dataset. Efficient methods include using a min-heap of size K, giving 
O(nlogK) time complexity, or Quick Select for average 
O(n) time."
---

# Interesting problems about sorting algorithm

## intro problem - Leetcode 215: Kth Largest Element in an Array

Given an integer array nums and an integer k, return the kth largest element in the array.

Note that it is the kth largest element in the sorted order, not the kth distinct element.

Can you solve it without sorting?

Example 1:

Input: nums = [3,2,1,5,6,4], k = 2
Output: 5
Example 2:

Input: nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4

## Quick sort

> the key idea is find the pivot, let the left number of the pivot is alway less than the pivot and the right number of the pivot is larger than the pivot.

```
func quickSort(nums []int, l, r int) {
    if l >= r {
        return
    }
    p := partition(nums, l , r)
    quickSort(nums, l, p-1)
    quickSort(nums, p+1, r)
}

func partition(nums []int, l , r int) int {
    index := l
    flag := nums[r]
    for i := l ; i < r ;i++ {
        if nums[i] < flag {
            nums[index], nums[i] = nums[i] , nums[index]
            index++
        } 
    }
    nums[index], nums[r] = nums[r], nums[index]
    return index
}
```

### Time Complexity of Quicksort

Quicksort is a highly efficient sorting algorithm and is based on the divide-and-conquer approach. Its performance can vary depending on the choice of the pivot element and the input data. The time complexity analysis of Quicksort involves examining the best, worst, and average cases.

Best Case Time Complexity: O(n log n)
In the best case, the pivot always divides the array into two equal halves. This ensures that the depth of the recursion tree is
logn and at each level of recursion, the entire array is processed.

Worst Case Time Complexity: O(n^2)
In the worst case, the pivot divides the array such that one part has only one element, and the other part has n−1 elements. This happens when the smallest or largest element is always picked as the pivot, leading to highly unbalanced partitions.

### Concurrent Quicksort with goroutines in Go

firstly, I came up with the idea the use goroutine to accelerate the calculation speed. but the version has some problems, can u find it?

```
package main

import (
 "fmt"
 "sync"
)

func main() {
 a := []int{3, 2, 1, 5, 6, 4}
 quickSortV2(a, 0, len(a)-1)
 fmt.Println(a)
}

func quickSortV2(nums []int, l, r int) {
 if l < r {
  var wg sync.WaitGroup
  p := partition(nums, l, r)
  wg.Add(2)
  go quickSortV2(nums, l, p-1)
  go quickSortV2(nums, p+1, r)
  wg.Wait()
 }
}

func partition(nums []int, l, r int) int {
 index := l
 flag := nums[r]
 for i := l; i < r; i++ {
  if nums[i] < flag {
   nums[index], nums[i] = nums[i], nums[index]
   index++
  }
 }
 nums[index], nums[r] = nums[r], nums[index]
 return index
}
```

In concurrent programming, `sync.WaitGroup` is an essential tool for waiting for a group of goroutines to complete their execution. The core functionality of the WaitGroup counter is to track the number of goroutines that are running by incrementing and decrementing the counter. When the counter reaches zero, it indicates that all goroutines have finished.

Reason: Why does `sync.WaitGroup` need to be passed in each recursive call?

Using a single WaitGroup to track all goroutines:

The core purpose of `sync.WaitGroup` is to wait for a set of goroutines to finish. By passing the same WaitGroup instance, all goroutines can use the same counter to correctly track the completion status of all goroutines.
If a new WaitGroup is created for each recursive call, these WaitGroups are independent of each other and cannot track the execution status of all goroutines as a whole. Therefore, the `wg.Wait()` in the main function might return before some sub-tasks are completed, as it is waiting for the initial WaitGroup passed to it, not the newly created ones.
Synchronization issues in recursive structures:

In recursive calls, it is necessary to wait for the sorting of both the left and right sub-arrays to be completed. If different WaitGroups are used, it is not possible to synchronize these sub-tasks.
Passing the same WaitGroup instance ensures that all goroutines at all levels of recursion are correctly tracked, regardless of which level of recursion they are on.

the right version:

```
package main

import (
 "fmt"
 "sync"
)

func main() {
 a := []int{3, 2, 1, 5, 6, 4}
 var wg sync.WaitGroup
 wg.Add(1) // 初始化计数器为1
 go quickSortV2(a, 0, len(a)-1, &wg)
 wg.Wait() // 等待所有 goroutine 完成
 fmt.Println(a)
}

func quickSortV2(nums []int, l, r int, wg *sync.WaitGroup) {
 defer wg.Done() // 确保函数结束时通知 WaitGroup 当前 goroutine 完成
 if l < r {
  p := partition(nums, l, r)
  wg.Add(2)                        // 增加计数器以跟踪新的 goroutine
  go quickSortV2(nums, l, p-1, wg) // 处理左子数组
  go quickSortV2(nums, p+1, r, wg) // 处理右子数组
 }
}

func partition(nums []int, l, r int) int {
 index := l
 flag := nums[r]
 for i := l; i < r; i++ {
  if nums[i] < flag {
   nums[index], nums[i] = nums[i], nums[index]
   index++
  }
 }
 nums[index], nums[r] = nums[r], nums[index]
 return index
}
```

### Concurrent Quicksort with Channels in Go

QuickSort Function:
Runs in a loop, continually checking for new tasks in taskChan.
If a task is received, and the subarray has more than one element (low < high), it performs partitioning to find the pivot.
Sends new tasks for the left and right subarrays into taskChan.
Uses a default case to check if taskChan is empty. If it is, it signals completion by sending true to doneChan and returns.

```
package main

import (
 "fmt"
)

type Task struct {
 low, high int
}

func main() {
 arr := []int{3, 2, 1, 5, 6, 4}
 taskChan := make(chan Task, len(arr))
 doneChan := make(chan bool)

 // Start the sorting goroutine
 go quickSort(arr, taskChan, doneChan)

 // Initial task to sort the whole array
 taskChan <- Task{0, len(arr) - 1}

 // Wait for sorting to complete
 <-doneChan
 fmt.Println(arr)
}

func quickSort(arr []int, taskChan chan Task, doneChan chan bool) {
 for {
  select {
  case task := <-taskChan:
   if task.low < task.high {
    p := partition(arr, task.low, task.high)
    // Send new tasks to channel
    taskChan <- Task{task.low, p - 1}
    taskChan <- Task{p + 1, task.high}
   }
  default:
   if len(taskChan) == 0 {
    // No more tasks left to process
    doneChan <- true
    return
   }
  }
 }
}

func partition(arr []int, low, high int) int {
 pivot := arr[high]
 i := low - 1
 for j := low; j < high; j++ {
  if arr[j] < pivot {
   i++
   arr[i], arr[j] = arr[j], arr[i]
  }
 }
 arr[i+1], arr[high] = arr[high], arr[i+1]
 return i + 1
}
```

This implementation leverages Go's concurrency model to efficiently sort an array using the quicksort algorithm. By using channels to manage tasks and synchronization, we can concurrently process different subarrays, potentially speeding up the sorting process on multi-core systems.

## Quick select (Solution of Kth Largest Element)

> the pivot location is settled, once it is calculated.
similar like binary sort, if p > k, the result must be in the left part of p, else if p < k, the result must be in the right part of p.

```
func quickSelect(nums []int, l, r , k int) int {
    p := partition(nums, l , r)
    // fmt.Println(l ,r , p)
    if p > k  {
        return quickSelect(nums, l, p-1, k)
    } else if p < k {
        return quickSelect(nums, p+1, r, k)
    } else {
        return nums[p]
    }
}
```

Quickselect is an algorithm used to find the k-th smallest (or largest) element in an unordered list. It is related to the quicksort algorithm. The key idea behind quickselect is similar to that of quicksort: it uses a pivot to partition the array such that the pivot is in its final sorted position, and all elements to the left of the pivot are smaller, and all elements to the right are larger. However, unlike quicksort, quickselect only recursively processes one side of the pivot, making it more efficient for this specific task.

Best Case: O(n)

In the best case, each pivot selected partitions the array into two nearly equal parts. On average, this reduces the problem size by about half each time.
The recurrence relation in this case is T(n) = T(n/2) + O(n), where T(n) is the time complexity for a list of size n.
This recurrence solves to O(n).

Average Case: O(n)

On average, the pivot chosen will partition the array such that each recursive step reduces the problem size significantly.
The same recurrence relation T(n) = T(n/2) + O(n) holds, leading to an average time complexity of O(n).

Worst Case: O(n^2)

In the worst case, the pivot chosen is always the smallest or largest element, resulting in the partition step reducing the problem size by only one element.
The recurrence relation in this case is T(n) = T(n-1) + O(n), leading to T(n) = O(n) + O(n-1) + ... + O(1) = O(n^2).
The worst-case scenario can be mitigated by using strategies such as random pivot selection or the Median of Medians method to ensure a more balanced partition.

The worst-case performance can often be avoided with good pivot selection strategies, making quickselect a practical choice for finding the k-th smallest (or largest) element in an array.

## Heap sort (Solution of Kth Largest Element)

here is also a problem, `maxHeap` the first function I use can not build max heap (this function can only find the top one), `maxHeapV2` can guarantee the result is a max heap.

```
func findKthLargest(nums []int, k int) int {
 maxHeapV2(nums, len(nums))
    fmt.Println(nums)
 for j := 1; j < k; j++ {
       
        nums[0], nums[len(nums)-j] = nums[len(nums)-j], nums[0]
        //  fmt.Println(nums)
        siftDown(nums, 0, len(nums)-j)
    }
    fmt.Println(nums)
    return nums[0]
}

func maxHeap(nums []int, size int) {
    for i := size -1 ; i > 0; i-- {
        targetIndex := (i-1) / 2
        if nums[i] > nums[targetIndex] {
            nums[i] , nums[targetIndex] = nums[targetIndex], nums[i]
        }
    }
}

func maxHeapV2(nums []int, size int) {
    for i := (size / 2) - 1; i >= 0; i-- {
        siftDown(nums, i, size)
    }
}
func siftDown(nums []int, i int, size int) {
    largest := i
    left := 2*i + 1
    right := 2*i + 2

    if left < size && nums[left] > nums[largest] {
        largest = left
    }
    if right < size && nums[right] > nums[largest] {
        largest = right
    }
    if largest != i {
        nums[i], nums[largest] = nums[largest], nums[i]
        siftDown(nums, largest, size)
    }
}
```
