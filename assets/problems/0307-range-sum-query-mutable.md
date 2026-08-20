---
id: 307
title: "Range Sum Query - Mutable"
slug: range-sum-query-mutable
difficulty: Medium
tags: [Array, Divide and Conquer, Design, Binary Indexed Tree, Segment Tree]
"@local": [Operator]
neetcode150_category: null
blind75_category: null
date_solved: 2026-07-29
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Range Sum Query - Mutable

Given an integer array `nums`, handle multiple queries of the following types:

1.  **Update** the value of an element in `nums`.
2.  Calculate the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** where `left <= right`.

Implement the `NumArray` class:

*   `NumArray(int[] nums)` Initializes the object with the integer array `nums`.
*   `void update(int index, int val)` **Updates** the value of `nums[index]` to be `val`.
*   `int sumRange(int left, int right)` Returns the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** (i.e. `nums[left] + nums[left + 1] + ... + nums[right]`).

**Example 1:**

**Input**
\["NumArray", "sumRange", "update", "sumRange"\]
\[\[\[1, 3, 5\]\], \[0, 2\], \[1, 2\], \[0, 2\]\]
**Output**
\[null, 9, null, 8\]

**Explanation**
NumArray numArray = new NumArray(\[1, 3, 5\]);
numArray.sumRange(0, 2); // return 1 + 3 + 5 = 9
numArray.update(1, 2);   // nums = \[1, 2, 5\]
numArray.sumRange(0, 2); // return 1 + 2 + 5 = 8

**Constraints:**

*   `1 <= nums.length <= 3 * 104`
*   `-100 <= nums[i] <= 100`
*   `0 <= index < nums.length`
*   `-100 <= val <= 100`
*   `0 <= left <= right < nums.length`
*   At most `3 * 104` calls will be made to `update` and `sumRange`.

## Code Template

### Go
```go
type NumArray struct {
    
}


func Constructor(nums []int) NumArray {
    
}


func (this *NumArray) Update(index int, val int)  {
    
}


func (this *NumArray) SumRange(left int, right int) int {
    
}


/**
 * Your NumArray object will be instantiated and called as such:
 * obj := Constructor(nums);
 * obj.Update(index,val);
 * param_2 := obj.SumRange(left,right);
 */
```

### TypeScript
```typescript
class NumArray {
    constructor(nums: number[]) {
        
    }

    update(index: number, val: number): void {
        
    }

    sumRange(left: number, right: number): number {
        
    }
}

/**
 * Your NumArray object will be instantiated and called as such:
 * var obj = new NumArray(nums)
 * obj.update(index,val)
 * var param_2 = obj.sumRange(left,right)
 */
```
