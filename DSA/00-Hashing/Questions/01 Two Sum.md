# 1. Two Sum

## Problem Statement
Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

You can return the answer in any order.

## Examples
**Example 1:**
- Input: nums = [2,7,11,15], target = 9
- Output: [0,1]
- Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].

**Example 2:**
- Input: nums = [3,2,4], target = 6
- Output: [1,2]

**Example 3:**
- Input: nums = [3,3], target = 6
- Output: [0,1]

## Constraints
- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- Only one valid answer exists.

Follow-up: Can you come up with an algorithm that is less than O(n^2) time complexity?

---

# understanding the question 

Pointing out these things:
- i. **Draw examples:** If `nums = [2, 7, 11, 15]` and `target = 9`, we need to find two numbers that sum to 9. 2 + 7 = 9. Their indices are 0 and 1.
- ii. **Clarify edge cases:** Array could have negative numbers. We are guaranteed exactly one valid solution, so we don't need to handle "no solution" cases.
- iii. **Confirm input/output:** Input is an array of integers and a target integer. Output is an array of two integers (the indices). 
- iv. **Important key words for the approach:** "exactly one solution", "indices", "add up to target". Since we need indices, sorting the array directly might mess up the original indices unless we store them. 
- v. **basic level of understanding:** We just need to find a pair `(x, y)` such that `x + y = target`. Equivalently, for every `x`, we need to find if `target - x` exists in the array.

# understanding the constraints

Pointing out these things: 
- i. **Time complexity:** $O(n^2)$ is trivial. For $N = 10^4$, $O(N^2)$ is $10^8$ operations, which might just barely pass or be slow. We want $O(N)$ or $O(N \log N)$.
- ii. **Space complexity:** We can afford $O(N)$ space since $N = 10^4$ is small enough to fit in memory easily.
- iii. **Input space, output space:** $N \le 10^4$, integers fit in standard 32-bit signed integer. 
- iv. **What kind of data structure or algorithm can be used here:** Hashing (HashMap) allows $O(1)$ lookups. If we use a HashMap to store the numbers we've seen and their indices, we can find the complement `target - x` in $O(1)$ time. 
- v. **how constraints help us to find the solution:** The $10^4$ length pushes us to find a better solution than $O(N^2)$. The fact that we need to return indices means a HashMap mapping `value -> index` is perfect.

# Solution 

## Brute force 

- **Intuition for the brute force:** For every element, check every other element that comes after it to see if they sum up to the target.
- **pseudo code for the brute force:**
  ```java
  for i from 0 to n-1:
      for j from i+1 to n-1:
          if nums[i] + nums[j] == target:
              return [i, j]
  ```
- **draw the dry run for the brute force:** 
  `nums = [3, 2, 4], target = 6`
  - i=0 (3): j=1 (2) -> 3+2=5 != 6. j=2 (4) -> 3+4=7 != 6
  - i=1 (2): j=2 (4) -> 2+4=6 == 6. Return [1, 2]
- **Time complexity and space complexity:** Time is $O(N^2)$, Space is $O(1)$.
- **solution code:**
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[] {i, j};
                }
            }
        }
        return new int[] {};
    }
}
```

## optimised code (using Hashing)

- **how we are optimising from the brute force:** Instead of a nested loop to find the complement (`target - nums[i]`), we can use a HashMap to remember the elements we've seen so far.
- **Intuition:** As we iterate through the array, we check if the complement (`target - nums[i]`) is already in our HashMap. If it is, we found our pair! If not, we add the current number and its index to the HashMap for future lookups.
- **pseudo code for the optimised approach:**
  ```java
  HashMap map
  for i from 0 to n-1:
      complement = target - nums[i]
      if map contains complement:
          return [map.get(complement), i]
      map.put(nums[i], i)
  ```
- **draw the dry run for the optimised approach:**
  `nums = [3, 2, 4], target = 6`
  - map = {}
  - i=0, num=3, complement=6-3=3. map doesn't contain 3. map.put(3, 0) => {3:0}
  - i=1, num=2, complement=6-2=4. map doesn't contain 4. map.put(2, 1) => {3:0, 2:1}
  - i=2, num=4, complement=6-4=2. map contains 2! Return [map.get(2), 2] -> [1, 2]
- **Time complexity and space complexity:** Time is $O(N)$ because we traverse the list exactly once, and each lookup in the table costs $O(1)$ time. Space is $O(N)$ for the hash map to store at most $N$ elements.
- **solution code:**
```java
import java.util.HashMap;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            if (map.containsKey(complement)) {
                return new int[] {map.get(complement), i};
            }
            
            map.put(nums[i], i);
        }
        
        return new int[] {}; // Should not reach here based on constraints
    }
}
```

# question where I went wrong & what is the correction 
*(No actual mistakes recorded for this run since we skipped directly to the solution, but typical pitfalls include:)*
- Trying to sort the array first but losing track of the original indices. (Correction: If sorting is used, store pairs of `(value, original_index)`).
- Adding the current element to the HashMap *before* checking for the complement, which could cause you to use the same element twice (e.g., target 6, array `[3, 3]`, picking `3` at index 0 twice). (Correction: Check for the complement first, then add the current element).
