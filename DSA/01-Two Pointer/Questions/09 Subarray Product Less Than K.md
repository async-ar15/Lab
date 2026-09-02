# 713. Subarray Product Less Than K

Given an array of integers `nums` and an integer `k`, return *the number of contiguous subarrays where the product of all the elements in the subarray is strictly less than* `k`.

### Example 1:
**Input:** nums = [10,5,2,6], k = 100
**Output:** 8
**Explanation:** The 8 subarrays that have product less than 100 are:
[10], [5], [2], [6], [10, 5], [5, 2], [2, 6], [5, 2, 6]
Note that [10, 5, 2] is not included as the product of 100 is not strictly less than k.

### Example 2:
**Input:** nums = [1,2,3], k = 0
**Output:** 0

### Constraints:
- `1 <= nums.length <= 3 * 10^4`
- `1 <= nums[i] <= 1000`
- `0 <= k <= 10^6`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** Array `[10, 5, 2, 6]` with `k = 100`. Valid subarrays are things like `[10]` (10 < 100), `[5, 2]` (10 < 100). 
- **Clarify edge cases:** If `k = 0`, since all numbers in the array are at least `1`, we can never have a product strictly less than 0. The same goes for `k = 1` (minimum product is 1, which is not *strictly less* than 1). So if `k <= 1`, we can instantly return 0.
- **Confirm input/output:** 
  - Input: Array `nums`, integer `k`.
  - Output: Integer count of valid subarrays.
- **Important keywords:** "contiguous subarrays", "strictly less".
- **Basic understanding:** The key word is **contiguous**. We are looking for blocks of numbers sitting right next to each other in the array. This means we absolutely **cannot sort the array**, because sorting would destroy the original contiguous blocks!

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** `nums.length <= 30,000`. An $O(n^2)$ algorithm would take around $30,000^2 = 900,000,000$ operations. This will likely give a Time Limit Exceeded (TLE) error. We need to find an $O(n)$ or $O(n \log n)$ solution.
- **Space complexity:** We just need to return a count, so $O(1)$ space.
- **What algorithm to use:** $O(n)$ Time + Contiguous Subarrays = **Sliding Window Pattern**.

# Solution 

## Brute Force (Two Nested Loops)

- **Intuition:** The easiest way to check every single contiguous subarray is to use two loops. The outer loop picks the starting number, and the inner loop expands the subarray to the right one by one, keeping a running product. Once the product bursts past `k`, we stop expanding and move the starting point forward.
- **Pseudo code:**
```text
SET count = 0
FOR start from 0 to n-1:
    SET product = 1
    FOR end from start to n-1:
        product = product * nums[end]
        IF product < k:
            count++
        ELSE:
            BREAK // Product is too big, stop expanding this starting point
RETURN count
```
- **Time Complexity:** $O(n^2)$ because of the nested loops. Will likely TLE.
- **Space Complexity:** $O(1)$. 

## Optimized Code (Sliding Window)

- **Intuition:** Instead of starting over for every single number, we can use a **Sliding Window**. Both `left` and `right` pointers start at index 0. The `right` pointer marches forward to expand the window, multiplying numbers into a `runningProduct`. 
  If the `runningProduct` gets too big (`>= k`), the window is invalid! We must shrink the window by moving the `left` pointer forward, dividing out the numbers we are leaving behind, until the product is strictly `< k` again.
  **The Math Trick:** Once the window is valid, how many new subarrays did we just find? Exactly `right - left + 1`. (For example, if the window is `[5, 2, 6]`, the new subarrays ending at 6 are `[6]`, `[2, 6]`, and `[5, 2, 6]`. That's 3 subarrays, which equals `right(2) - left(0) + 1 = 3`).
- **Pseudo code:**
```text
1. IF k <= 1, RETURN 0.
2. SET count = 0, runningProduct = 1, left = 0

3. FOR right from 0 to nums.length - 1:
4.     runningProduct = runningProduct * nums[right]
5.     
6.     WHILE runningProduct >= k:
7.         runningProduct = runningProduct / nums[left]
8.         left++
9.         
10.    count = count + (right - left + 1)
11.    
12. RETURN count
```
- **Time Complexity:** $O(n)$. Both `left` and `right` pointers only travel from left to right exactly once.
- **Space Complexity:** $O(1)$.
- **Solution Code (Java):**
```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        // Edge case: Since all numbers are >= 1, we can never have a product < 1.
        if (k <= 1) return 0;
        
        int count = 0;
        int runningProduct = 1;
        int left = 0;
        
        for (int right = 0; right < nums.length; right++) {
            // Expand the window
            runningProduct *= nums[right];
            
            // Shrink the window if the product gets too big
            while (runningProduct >= k) {
                runningProduct /= nums[left];
                left++;
            }
            
            // Math trick: The number of valid contiguous subarrays ending at 'right' 
            // is exactly equal to the size of the window!
            count += (right - left + 1);
        }
        
        return count;
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

1. **Trying to sort the array:**
   - *My mistake:* Assuming I should sort the array because we wanted numbers less than `k`.
   - *Correction:* The question explicitly asks for **contiguous subarrays**. Sorting the array destroys the original contiguous order of the elements. You can almost never sort the array if the problem asks for subarrays (subsequences are fine, but not subarrays).

2. **Not understanding the Sliding Window math:**
   - *My mistake:* Getting completely stuck on how to count the subarrays without manually looping through them all. 
   - *Correction:* The magic formula is `count += (right - left + 1)`. If you have a valid window of elements, the number of new valid subarrays that end at the `right` pointer is exactly equal to the size of the window!

3. **Java Syntax Struggles:**
   - *My mistake:* Getting lost in how to actually type out the `for` loops and `while` loops in Java (`while(subarr < 1) { left++ }`).
   - *Correction:* A `for` loop is perfect for the `right` pointer because it automatically increments by 1 every cycle (`for (int right = 0; right < nums.length; right++)`). Inside that loop, we use a simple `while (runningProduct >= k)` to shrink the `left` pointer as many times as necessary. Keep practicing the syntax, it will become muscle memory soon enough!