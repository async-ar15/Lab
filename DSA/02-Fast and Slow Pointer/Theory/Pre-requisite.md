# Fast and Slow Pointer: Pre-requisites

Before diving into the Fast & Slow pointer pattern, you absolutely must understand the data structure it is most commonly used on: **Linked Lists**. While this pattern can occasionally be used on arrays, 95% of the time, it's a Linked List trick.

> [!NOTE]
> **Prerequisite Reminder**
> We have already covered the fundamentals of Linked Lists (Nodes, Traversing, and Null Pointers) extensively in the previous module. 
> Please review [01-Two Pointer/Theory/Pre-requisite.md](../../01-Two%20Pointer/Theory/Pre-requisite.md) if you need a refresher!

## 1. Why Arrays are Sometimes Treated like Linked Lists
Sometimes, a problem will give you an Array, but you actually treat it like a Linked List! 
How? By using the *value* of the array as the *index* for the next jump.
For example, if `nums = [2, 0, 1]`:
- Start at index `0`. Value is `2`. So jump to index `2`.
- At index `2`, Value is `1`. So jump to index `1`.
This is exactly how Linked List jumping works, and it's a very common trick for cycle-detection problems!
