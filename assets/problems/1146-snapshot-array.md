---
id: 1146
title: "Snapshot Array"
slug: snapshot-array
difficulty: Medium
tags: [Array, Hash Table, Binary Search, Design]
neetcode150_category: null
blind75_category: null
date_solved: null
languages: [golang, typescript]
time_complexity: null
space_complexity: null
---

# Snapshot Array

Implement a SnapshotArray that supports the following interface:

*   `SnapshotArray(int length)` initializes an array-like data structure with the given length. **Initially, each element equals 0**.
*   `void set(index, val)` sets the element at the given `index` to be equal to `val`.
*   `int snap()` takes a snapshot of the array and returns the `snap_id`: the total number of times we called `snap()` minus `1`.
*   `int get(index, snap_id)` returns the value at the given `index`, at the time we took the snapshot with the given `snap_id`

**Example 1:**

**Input:** \["SnapshotArray","set","snap","set","get"\]
\[\[3\],\[0,5\],\[\],\[0,6\],\[0,0\]\]
**Output:** \[null,null,0,null,5\]
**Explanation:** 
SnapshotArray snapshotArr = new SnapshotArray(3); // set the length to be 3
snapshotArr.set(0,5);  // Set array\[0\] = 5
snapshotArr.snap();  // Take a snapshot, return snap\_id = 0
snapshotArr.set(0,6);
snapshotArr.get(0,0);  // Get the value of array\[0\] with snap\_id = 0, return 5

**Constraints:**

*   `1 <= length <= 5 * 104`
*   `0 <= index < length`
*   `0 <= val <= 109`
*   `0 <= snap_id <` (the total number of times we call `snap()`)
*   At most `5 * 104` calls will be made to `set`, `snap`, and `get`.

## Code Template

### Go
```go
type SnapshotArray struct {
    
}


func Constructor(length int) SnapshotArray {
    
}


func (this *SnapshotArray) Set(index int, val int)  {
    
}


func (this *SnapshotArray) Snap() int {
    
}


func (this *SnapshotArray) Get(index int, snap_id int) int {
    
}


/**
 * Your SnapshotArray object will be instantiated and called as such:
 * obj := Constructor(length);
 * obj.Set(index,val);
 * param_2 := obj.Snap();
 * param_3 := obj.Get(index,snap_id);
 */
```

### TypeScript
```typescript
class SnapshotArray {
    constructor(length: number) {
        
    }

    set(index: number, val: number): void {
        
    }

    snap(): number {
        
    }

    get(index: number, snap_id: number): number {
        
    }
}

/**
 * Your SnapshotArray object will be instantiated and called as such:
 * var obj = new SnapshotArray(length)
 * obj.set(index,val)
 * var param_2 = obj.snap()
 * var param_3 = obj.get(index,snap_id)
 */
```
