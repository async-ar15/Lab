# 16. 3Sum Closest

Given an integer array `nums` of length `n` and an integer `target`, find three integers in `nums` such that the sum is closest to `target`.

Return *the sum of the three integers*.

You may assume that each input would have exactly one solution.

### Example 1:
**Input:** nums = [-1,2,1,-4], target = 1
**Output:** 2
**Explanation:** The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).

### Example 2:
**Input:** nums = [0,0,0], target = 1
**Output:** 0
**Explanation:** The sum that is closest to the target is 0. (0 + 0 + 0 = 0).

### Constraints:
- `3 <= nums.length <= 500`
- `-1000 <= nums[i] <= 1000`
- `-10^4 <= target <= 10^4`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** If `nums = [-1, 2, 1, -4]` and `target = 1`, the closest we can get is `2` by summing `-1 + 2 + 1`. 
- **Clarify edge cases:** The constraints guarantee `nums.length >= 3`, so we will always have at least one valid triplet.
- **Confirm input/output:** 
  - Input: An array of integers `nums` and an integer `target`.
  - Output: The *actual sum* of the three integers (not the difference, not the integers themselves, just the sum).
- **Important keywords:** "three integers", "closest to target", "exactly one solution".
- **Basic understanding:** This is identical to the `3Sum` problem, except instead of looking for exactly `0`, we are looking for a sum that gets as close to `target` as possible. Since it's a 3Sum variant, we should sort the array and use Two Pointers. The only tricky part is the "math" of tracking the closest sum.

# Understanding the Constraints

- **Time complexity:** `nums.length <= 500`. This is a very small constraint! An $O(n^3)$ brute force would take $500^3 = 125,000,000$ operations, which *might* barely pass but is very risky. An $O(n^2)$ algorithm will take $500^2 = 250,000$ operations, which is lightning fast. We definitely want the $O(n^2)$ approach.
- **Space complexity:** We just need to track a single integer (`closestSum`), so $O(1)$ space.
- **What algorithm to use:** $O(n^2)$ Time + Triplets = **Sorting + Two Pointers**.

# Solution 

## Brute Force (Three Loops)

- **Intuition:** Use 3 nested loops to check every single combination of 3 numbers. For each combination, check how far its sum is from the target. If it's closer than the previous closest, update the record.
- **Pseudo code:**
```text
SET closestSum = Infinity
FOR i from 0 to n-1:
    FOR j from i+1 to n-1:
        FOR k from j+1 to n-1:
            SET sum = nums[i] + nums[j] + nums[k]
            IF absolute_difference(sum, target) < absolute_difference(closestSum, target):
                closestSum = sum
RETURN closestSum
```
- **Time Complexity:** $O(n^3)$. 
- **Space Complexity:** $O(1)$. 
- **Thoughts:** Too slow for a real interview. We can do better using the standard 3Sum sorting trick.

## Optimized Code (Sorting + Two Pointers)

- **Intuition:** Just like regular `3Sum`, we sort the array and use a `for` loop to lock in the first number (`nums[i]`). Then we use `left` and `right` pointers for the rest of the array. 
  
  **The Mathematics trick:** How do we define "closest"? The closest sum is the one where the *absolute difference* between the `sum` and the `target` is the smallest. We can use `Math.abs(sum - target)` to check the distance.
  - If `sum < target`, we are too small. We move `left++` to make the sum bigger.
  - If `sum > target`, we are too big. We move `right--` to make the sum smaller.
  - If `sum == target`, the distance is `0`. We can't get any closer than that, so we can immediately return!
- **Pseudo code:**
```text
1. Sort the nums array
2. SET closestSum = nums[0] + nums[1] + nums[2] (initialize with the first valid triplet)

3. FOR i from 0 to length - 1:
4.     SET left = i + 1
5.     SET right = length - 1
6.    
7.     WHILE left < right:
8.         SET currentSum = nums[i] + nums[left] + nums[right]
9.         
10.        // The Math check: Is this sum closer to the target than our recorded closestSum?
11.        IF Math.abs(currentSum - target) < Math.abs(closestSum - target):
12.            closestSum = currentSum
13.            
14.        // Standard Two Pointer movement
15.        IF currentSum > target: right--
16.        ELSE IF currentSum < target: left++
17.        ELSE: RETURN currentSum // Distance is 0, perfect match!

18. RETURN closestSum
```
- **Time Complexity:** $O(n \log n)$ to sort + $O(n^2)$ for the loops = **$O(n^2)$** overall time.
- **Space Complexity:** $O(1)$ auxiliary space.
- **Solution Code (Java):**
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        
        // Initialize with the sum of the first three elements
        int closestSum = nums[0] + nums[1] + nums[2];
        
        for (int i = 0; i < nums.length; i++) {
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) {
                int currentSum = nums[i] + nums[left] + nums[right];
                
                // The Mathematics: checking the absolute distance to the target
                if (Math.abs(currentSum - target) < Math.abs(closestSum - target)) {
                    closestSum = currentSum;
                }
                
                // Move pointers based on how currentSum compares to the target
                if (currentSum > target) {
                    right--;
                } else if (currentSum < target) {
                    left++;
                } else {
                    // If currentSum == target, distance is 0. We found the perfect answer.
                    return currentSum;
                }
            }
        }
        
        return closestSum;
    }
}
```

---

# Mistakes & Corrections

The main traps for this problem revolve around the mathematics:

1. **Trying to initialize `closestSum` to `Integer.MAX_VALUE`:**
   - *My mistake:* Writing `int closestSum = Integer.MAX_VALUE;` and then checking `Math.abs(closestSum - target)`.
   - *Correction:* If `closestSum` is `Integer.MAX_VALUE` and the `target` is negative, doing `closestSum - target` can cause an integer overflow, resulting in a negative number, which completely breaks the `Math.abs` logic. Always initialize `closestSum` to an actual valid sum from the array (like `nums[0] + nums[1] + nums[2]`).

2. **Returning the distance instead of the sum:**
   - *My mistake:* Returning `Math.abs(closestSum - target)`.
   - *Correction:* The question asks to return *the sum of the three integers*, not how close they were to the target! Just return `closestSum`.
