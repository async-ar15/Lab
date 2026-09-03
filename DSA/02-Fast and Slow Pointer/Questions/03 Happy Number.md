# 202. Happy Number

Write an algorithm to determine if a number `n` is happy.

A **happy number** is a number defined by the following process:
- Starting with any positive integer, replace the number by the sum of the squares of its digits.
- Repeat the process until the number equals 1 (where it will stay), or it **loops endlessly in a cycle** which does not include 1.
- Those numbers for which this process **ends in 1** are happy.

Return `true` *if* `n` *is a happy number, and* `false` *if not*.

### Example 1:
**Input:** n = 19
**Output:** true
**Explanation:**
$1^2 + 9^2 = 82$
$8^2 + 2^2 = 68$
$6^2 + 8^2 = 100$
$1^2 + 0^2 + 0^2 = 1$

### Example 2:
**Input:** n = 2
**Output:** false

### Constraints:
- `1 <= n <= 2^31 - 1`

---

# Understanding the Question 

**What is a Happy Number?**
A Happy Number is just a number that eventually turns into `1` when you repeatedly replace it with the sum of the squares of its digits.
For example, if you start with `19`:
1. The digits are `1` and `9`. 
2. Square them and add them up: $1^2 + 9^2 = 1 + 81 = 82$.
3. Now take `82`. The digits are `8` and `2`.
4. Square and add: $8^2 + 2^2 = 64 + 4 = 68$.
5. Now take `68`: $6^2 + 8^2 = 36 + 64 = 100$.
6. Now take `100`: $1^2 + 0^2 + 0^2 = 1$.
Because it eventually reached `1`, `19` is a **Happy Number**!

If a number is NOT happy, it will never reach `1`. Instead, it will get trapped in an endless loop of repeating numbers.

Breaking down the problem:
- **Draw examples:** `19` becomes `1` (true). `2` becomes `4`, `16`, `37`, `58`, `89`, `145`, `42`, `20`, `4`... Wait, `4` again! It's trapped in a loop! So `2` is (false).
- **Confirm input/output:** 
  - Input: Integer `n`.
  - Output: Boolean (`true` or `false`).
- **Important keywords:** "loops endlessly in a cycle".

# Understanding the Constraints

What the constraints are secretly telling me:
- **Space complexity:** If we want the absolute best solution, we should aim for $O(1)$ extra space.
- **The `2^31 - 1` Constraint:** This equals `2,147,483,647`. It is the absolute maximum positive value that a standard 32-bit `int` variable can hold in Java. Whenever you see this constraint, it is LeetCode's secret way of telling you: *"Don't worry about using `long`, a standard `int` is perfectly fine here, it will not overflow!"*

# Solution 

## Brute Force (HashSet)

- **Intuition:** If a number isn't happy, it gets trapped in a cycle. How do we detect cycles? We remember where we've been! Using a `HashSet` to store every number we generate is the perfect Brute Force approach. If we generate a number and it is already in our set, we immediately know we are in a loop and can return `false`.
- **Pseudo code:**
```text
SET visited = new HashSet

WHILE n != 1:
    IF visited contains n:
        RETURN false // We are in a loop!
    visited.add(n)
    n = getNext(n) // Helper function to do the math

RETURN true
```
- **Time Complexity:** $O(\log N)$ to process the digits of the numbers.
- **Space Complexity:** $O(\log N)$ to store the numbers in the HashSet.

## Optimized Code (Fast & Slow Pointers)

- **Intuition:** We are going to treat **math** like a Linked List! 
  Imagine every number is a Node. The "next" pointer is simply the math function that calculates the sum of the squares of the digits!
  Since we are just jumping from one number to the next, and we know we might fall into a cycle, we can use **Fast & Slow Pointers**!
  - `slow` calculates the next number ONCE.
  - `fast` calculates the next number TWICE (it calculates the next of the next).
  
  If `fast` ever hits `1`, it's a Happy Number! If `slow == fast`, they collided in a cycle, meaning it's NOT a Happy Number!
- **Pseudo code:**
```text
// We need a helper function first
FUNCTION getNext(n):
    totalSum = 0
    WHILE n > 0:
        digit = n % 10
        totalSum = totalSum + (digit * digit)
        n = n / 10
    RETURN totalSum

// Main Function
SET slow = n
SET fast = getNext(n)

WHILE fast != 1 AND slow != fast:
    slow = getNext(slow)            // 1 step
    fast = getNext(getNext(fast))   // 2 steps

IF fast == 1:
    RETURN true
ELSE:
    RETURN false
```
- **Time Complexity:** $O(\log N)$.
- **Space Complexity:** $O(1)$! We only use two pointers, no matter how long it takes to find the cycle.
- **Solution Code (Java):**
```java
class Solution {
    public boolean isHappy(int n) {
        int slow = n;
        // Fast starts one step ahead to prevent an immediate collision on the first check
        int fast = getNext(n);
        
        // Keep running until fast reaches 1 (happy) or they collide (sad loop)
        while (fast != 1 && slow != fast) {
            slow = getNext(slow);
            fast = getNext(getNext(fast));
        }
        
        return fast == 1;
    }
    
    // Helper function to extract digits and sum their squares
    private int getNext(int n) {
        int totalSum = 0;
        
        while (n > 0) {
            int d = n % 10;          // Extract the last digit
            n = n / 10;              // Chop off the last digit
            totalSum += d * d;       // Square it and add to total
        }
        
        return totalSum;
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

1. **Extracting Digits in Java:**
   - *My mistake:* Forgetting how to split a number into individual digits (like turning `82` into `8` and `2`).
   - *Correction:* Use the Modulo/Divide trick! 
     - `n % 10` gives you the absolute last digit of a number. (e.g., `82 % 10 = 2`).
     - `n / 10` chops off the last digit. (e.g., `82 / 10 = 8`).
     - Loop this until `n == 0`!

2. **Starting Fast and Slow:**
   - *My mistake:* Setting both `slow = n` and `fast = n` before the loop.
   - *Correction:* If you set them both to `n`, then your `while (slow != fast)` condition is instantly false, and the loop never even runs! You must give `fast` a head start by doing `int fast = getNext(n);` so they start at different values.



```java



```