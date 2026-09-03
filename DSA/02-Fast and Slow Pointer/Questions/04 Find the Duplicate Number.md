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

Breaking down the problem:
- **Draw examples:** If `nums = [1, 3, 4, 2, 2]`, the number `2` appears twice, so we return `2`.
- **Confirm input/output:** 
  - Input: Array `nums`.
  - Output: Integer (the duplicate number).
- **Important keywords:** "without modifying", "constant extra space".
- **Basic understanding:** The problem is simple to solve normally, but the restrictions make it extremely tricky. We cannot sort the array (that modifies it). We cannot use a HashSet (that takes $O(N)$ extra space). We are forced to use a pure pointer algorithm.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Pigeonhole Principle:** The follow-up asks how we can prove a duplicate exists. If we have $N$ boxes (the range $1$ to $n$) and $N+1$ pigeons (the items in the array), at least one box MUST contain more than one pigeon!
- **The Secret Linked List:** The constraints state that the array has `n+1` numbers, and every number is between `1` and `n`. This means **every single value in the array is a valid index inside the array**! Because of this, we can treat the array exactly like a Linked List, where the value at a certain index is the pointer to the next index.

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
  2. Once they collide, put a new pointer at the start of the array.
  3. Move both pointers at 1x speed. Where they meet is the duplicate number!
- **Pseudo code:**
```text
SET slow = nums[0]
SET fast = nums[0]

// Step 1: Find collision
DO:
    slow = nums[slow]           // 1 step (like slow.next)
    fast = nums[nums[fast]]     // 2 steps (like fast.next.next)
WHILE slow != fast

// Step 2: Find start of cycle
SET start = nums[0]
WHILE start != slow:
    start = nums[start]         // 1 step
    slow = nums[slow]           // 1 step

RETURN start
```
- **Time Complexity:** $O(N)$.
- **Space Complexity:** $O(1)$. 
- **Solution Code (Java):**
```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = nums[0];
        int fast = nums[0];
        
        // Step 1: Find the intersection point of the two runners.
        // We use a do-while loop because if we use a normal while(slow != fast),
        // it will immediately fail since they both start at nums[0]!
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);
        
        // Step 2: Find the "entrance" to the cycle.
        int start = nums[0];
        
        while (start != slow) {
            start = nums[start];
            slow = nums[slow];
        }
        
        return start;
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
   - *My mistake:* Writing `while (slow != fast)` and initializing them both to `nums[0]`. The loop never runs because they instantly equal each other!
   - *Correction:* In the previous problem (Happy Number), we fixed this by manually calculating the first step for `fast` before the loop. Here, the cleanest way to fix it is to use a `do { ... } while (slow != fast);` loop. This forces the pointers to take at least one jump before checking if they collide!

2. **Array vs Linked List syntax:**
   - *My mistake:* Forgetting how to "jump" in an array.
   - *Correction:* In a Linked List, a jump is `slow = slow.next`. When treating an array as a Linked List, a jump is `slow = nums[slow]`. A double jump is `fast = nums[nums[fast]]`.
