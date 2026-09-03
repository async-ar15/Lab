# 287. Find the Duplicate Number

Given an array of integers `nums` containing `n + 1` integers where each integer is in the range `[1, n]` inclusive.

There is only **one repeated number** in `nums`, return *this repeated number*.

You must solve the problem **without** modifying the array `nums` and using only constant extra space.

### Example 1:
**Input:** nums = [1,3,4,2,2]
**Output:** 2

### Example 2:
**Input:** nums = [3,1,3,4,2]
**Output:** 3

### Example 3:
**Input:** nums = [3,3,3,3,3]
**Output:** 3

### Constraints:
- `1 <= n <= 10^5`
- `nums.length == n + 1`
- `1 <= nums[i] <= n`
- All the integers in `nums` appear only **once** except for **precisely one integer** which appears **two or more times**.

**Follow up:**
- How can we prove that at least one duplicate number must exist in `nums`?
- Can you solve the problem in linear runtime complexity?

---

# Understanding the Question 

**My Understanding:**
Sometimes you get the pattern but are not able to solve the question because that pattern does not fit into the correct data structure. So sometimes it's fine to do the conversion—like here we treated the array as a linked list. Going through the example step-by-step is extremely important to actually see the conversion!

**Agent Feedback:**
This is a massive "level-up" realization! In interviews, they will deliberately try to trick you by handing you an Array when the solution requires a Graph or Linked List algorithm. 
Tracing a concrete example (like `1 -> 3 -> 2 -> 4 -> 2`) on a whiteboard is literally the only way to "see through the matrix" and realize it's a cycle detection problem. That was a brilliant learning to lock in!

Breaking down the problem:
- **Draw examples:** If `nums = [1, 3, 4, 2, 2]`, the number `2` appears twice, so we return `2`.
- **Confirm input/output:** 
  - Input: Array `nums`.
  - Output: Integer (the duplicate number).
- **Important keywords:** "without modifying", "constant extra space".

# Understanding the Constraints

What the constraints are secretly telling me:
- **Pigeonhole Principle:** The follow-up asks how we can prove a duplicate exists. If we have $N$ boxes (the range $1$ to $n$) and $N+1$ pigeons (the items in the array), at least one box MUST contain more than one pigeon!
- **The Secret Linked List:** The constraints state that the array has `n+1` numbers, and every number is between `1` and `n`. This means **every single value in the array is a valid index inside the array**! Because of this, we can treat the array exactly like a Linked List.

# Solution 

## Brute Force (Using a HashSet)

- **Intuition:** The easiest way to find a duplicate is to remember everything we've seen. We use a `HashSet`. If we check a number and it's already in the set, we found the duplicate!
- **Pseudo code:**
```text
SET visited = new HashSet

FOR EACH num in nums:
    IF visited contains num:
        RETURN num
    visited.add(num)
```
- **Time Complexity:** $O(N)$.
- **Space Complexity:** $O(N)$. This fails the "constant extra space" constraint!

## Optimized Code (Fast & Slow Pointers / Floyd's Algorithm)

- **Intuition:** Let's treat the array like a Linked List. We jump from node to node using `current = nums[current]`. 
  Because there is a duplicate number, multiple indices will "point" to the exact same destination index. This creates a **cycle**!
  Finding the duplicate number is exactly the same as finding the **Start of the Cycle** (which is what we did in Linked List Cycle II). 
  
  1. Use `slow` and `fast` pointers to find the collision point.
  2. Once they collide, reset `slow` to the start of the array (index 0).
  3. Move both pointers at 1x speed. Where they meet is the duplicate number!
- **Pseudo code / Dry Run (My Way):**
```text
int slow = 0; fast = 0;
do {
    slow = nums[slow];
    fast = nums[nums[fast]];
} while (slow != fast);

slow = 0;
while (slow != fast) {
    slow = nums[slow];
    fast = nums[fast];
}
return slow;
```
**Agent Feedback (Your Code):**
Your code is flawless. Starting at `0` instead of `nums[0]` is a fantastic optimization because the problem guarantees numbers are from `[1, n]`. Therefore, no value will ever equal `0`, meaning index `0` is mathematically guaranteed to be *outside* the cycle. It acts perfectly as the `head` of our Linked List!

- **Time Complexity:** $O(N)$.
- **Space Complexity:** $O(1)$. 
- **Solution Code (Java):**
```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0;
        int fast = 0;
        
        // Step 1: Find the intersection point of the two runners.
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);
        
        // Step 2: Find the "entrance" to the cycle.
        slow = 0;
        
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        
        return slow;
    }
}
```

# Things told by the instructor

1. Understand the problem
2. Devise a strategy (find edge cases)
3. Breakdown the problem if possible 
4. Write a pseudocode
5. Implement the solution 
6. Testing and debugging 
7. Optimize and review 

---

# Mistakes & Corrections

1. **How to start the Fast & Slow loop:**
   - *My mistake:* Writing `while (slow != fast)` and initializing them both to the same starting point. The loop never runs because they instantly equal each other!
   - *Correction:* The cleanest way to fix it is to use a `do { ... } while (slow != fast);` loop. This forces the pointers to take at least one jump before checking if they collide!

2. **Array vs Linked List syntax:**
   - *My mistake:* Forgetting how to "jump" in an array.
   - *Correction:* In a Linked List, a jump is `slow = slow.next`. When treating an array as a Linked List, a jump is `slow = nums[slow]`. A double jump is `fast = nums[nums[fast]]`.
