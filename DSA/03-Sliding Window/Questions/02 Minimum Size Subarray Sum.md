# 209. Minimum Size Subarray Sum

Given an array of positive integers `nums` and a positive integer `target`, return the **minimal length** of a **subarray** whose sum is greater than or equal to `target`. If there is no such subarray, return `0` instead.

### Example 1:
**Input:** target = 7, nums = [2,3,1,2,4,3]
**Output:** 2
**Explanation:** The subarray [4,3] has the minimal length under the problem constraint.

### Example 2:
**Input:** target = 4, nums = [1,4,4]
**Output:** 1

### Example 3:
**Input:** target = 11, nums = [1,1,1,1,1,1,1,1]
**Output:** 0

### Constraints:
- `1 <= target <= 10^9`
- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^4`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** `target = 7, nums = [2,3,1,2,4,3]`
  - `[2, 3, 1, 2]` sum is 8. Length is 4.
  - `[1, 2, 4]` sum is 7. Length is 3.
  - `[4, 3]` sum is 7. Length is 2.
  - The minimum length that hits the target is 2.
- **Confirm input/output:** 
  - Input: Target integer and an array of positive integers.
  - Output: Integer (the minimum length, or `0` if impossible).
- **Important keywords:** "subarray", "minimal length". The word "subarray" implies contiguous elements, which again points directly to the **Sliding Window** pattern.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** `nums.length <= 10^5`. An $O(N^2)$ solution will take $\approx 10^{10}$ operations, which will cause a Time Limit Exceeded (TLE). We MUST solve this in $O(N)$ or $O(N \log N)$.
- **Positive Integers Only:** The constraint `1 <= nums[i]` is a massive hint. Because all numbers are positive, adding numbers will *always* increase the sum, and removing numbers will *always* decrease the sum. This guarantees that a Sliding Window approach will work without any weird edge cases involving negative numbers!

# Solution 

## Brute Force (Nested Loops)

- **Intuition:** Try every single possible subarray. For every starting point `i`, keep adding elements `j` until the sum is `>= target`. If it is, check if the length `j - i + 1` is the smallest we've seen so far.
- **Pseudo code:**
```text
SET minLength = Infinity

FOR i from 0 to nums.length - 1:
    SET sum = 0
    FOR j from i to nums.length - 1:
        sum = sum + nums[j]
        IF sum >= target:
            minLength = MIN(minLength, j - i + 1)
            BREAK // No need to keep adding, we want the MINIMAL length

IF minLength == Infinity: RETURN 0
ELSE: RETURN minLength
```
- **Time Complexity:** $O(N^2)$.
- **Space Complexity:** $O(1)$.

## Optimized Code (Variable-Size Sliding Window)

- **Intuition:** In the previous problem, the window size `k` was fixed. Here, we don't know the window size, we only know the `target` sum. This is a **Variable-Size Sliding Window**.
  
  The strategy is:
  1. Keep expanding the window by moving `right` pointer and adding to the sum until we hit the target.
  2. Once we hit the target, we record the window size.
  3. Now, try to **shrink** the window from the `left` side! Subtract `nums[left]` and move `left` forward. 
  4. If the sum is STILL `>= target` even after shrinking, record the new (smaller) window size! Keep shrinking until the sum falls below the target, then go back to expanding `right`.
- **Pseudo code:**
```text
SET minLength = Infinity
SET sum = 0
SET left = 0

FOR right from 0 to nums.length - 1:
    // Expand the window
    sum = sum + nums[right]
    
    // While the window is valid, try to shrink it
    WHILE sum >= target:
        minLength = MIN(minLength, right - left + 1)
        sum = sum - nums[left]
        left = left + 1

IF minLength == Infinity: RETURN 0
ELSE: RETURN minLength
```
- **Time Complexity:** $O(N)$. Even though there is a `while` loop inside a `for` loop, both the `left` and `right` pointers only ever move forward. Each element is added to the window exactly once and removed from the window exactly once. $2N$ operations = $O(N)$.
- **Space Complexity:** $O(1)$. 

- **Solution Code (Java):**
```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        // Initialize to MAX_VALUE so our Math.min check works correctly
        int minLength = Integer.MAX_VALUE;
        int sum = 0;
        int left = 0;
        
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right]; // Expand window by adding the right element
            
            // As long as the window meets the condition, try to shrink it from the left
            while (sum >= target) {
                // Record the current window's length
                minLength = Math.min(minLength, right - left + 1);
                
                // Shrink the window
                sum -= nums[left];
                left++;
            }
        }
        
        // If minLength never changed, we never hit the target, so return 0
        return minLength == Integer.MAX_VALUE ? 0 : minLength;
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

1. **Using `if` instead of `while` to shrink:**
   - *Common Mistake:* Writing `if (sum >= target)` to shrink the window.
   - *Correction:* You MUST use a `while` loop! Imagine `target = 7` and window is `[1, 1, 1, 100]`. If you only shrink once using an `if`, you get `[1, 1, 100]` which is length 3. But you need to keep shrinking until the sum falls below 7! A `while` loop correctly shrinks it all the way down to just `[100]` (length 1).

2. **The "Impossible" Edge Case:**
   - *Common Mistake:* Forgetting to check if `minLength` was ever actually updated, and just returning `minLength` at the end. If the array is `[1, 1, 1]` and `target = 100`, it will return `2147483647` (Integer.MAX_VALUE).
   - *Correction:* Always use a ternary operator or an if-statement at the end: `return minLength == Integer.MAX_VALUE ? 0 : minLength;`
