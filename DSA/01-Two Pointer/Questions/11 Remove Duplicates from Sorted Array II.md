# 80. Remove Duplicates from Sorted Array II

Given an integer array `nums` sorted in **non-decreasing order**, remove some duplicates **in-place** such that each unique element appears **at most twice**. The **relative order** of the elements should be kept the **same**.

Since it is impossible to change the length of the array in some languages, you must instead have the result be placed in the **first part** of the array `nums`. More formally, if there are `k` elements after removing the duplicates, then the first `k` elements of `nums` should hold the final result. It does not matter what you leave beyond the first `k` elements.

Return `k` *after placing the final result in the first* `k` *slots of* `nums`.

Do **not** allocate extra space for another array. You must do this by **modifying the input array in-place** with $O(1)$ extra memory.

### Example 1:
**Input:** nums = [1,1,1,2,2,3]
**Output:** 5, nums = [1,1,2,2,3,_]
**Explanation:** Your function should return k = 5, with the first five elements of nums being 1, 1, 2, 2 and 3 respectively.
It does not matter what you leave beyond the returned k (hence they are underscores).

### Example 2:
**Input:** nums = [0,0,1,1,1,1,2,3,3]
**Output:** 7, nums = [0,0,1,1,2,3,3,_,_]
**Explanation:** Your function should return k = 7, with the first seven elements of nums being 0, 0, 1, 1, 2, 3 and 3 respectively.
It does not matter what you leave beyond the returned k (hence they are underscores).

### Constraints:
- `1 <= nums.length <= 3 * 10^4`
- `-10^4 <= nums[i] <= 10^4`
- `nums` is sorted in non-decreasing order.

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** `[1, 1, 1, 2]` -> The third `1` is invalid. We want `[1, 1, 2]`. Return `k = 3`.
- **Clarify edge cases:** If the array length is 1 or 2, we can just return the length immediately, because it's impossible to have an element appear more than twice!
- **Confirm input/output:** 
  - Input: Sorted array `nums`.
  - Output: Integer `k` (the number of valid elements), but we must physically modify the `nums` array to have those `k` elements at the very beginning.
- **Important keywords:** "sorted", "at most twice", "in-place", "$O(1)$ extra memory".
- **Basic understanding:** We need to scan the array and keep numbers, but as soon as we see a number for the 3rd time, we ignore it. Because it's in-place, we'll need Two Pointers: one to scan, one to place the valid numbers.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n)$ is expected.
- **Space complexity:** Explicitly restricted to $O(1)$ memory. This immediately rules out using a `HashMap` or a `Frequency Array` to count occurrences!
- **What algorithm to use:** $O(1)$ Space + In-Place Array Modification = **Two Pointers**. The fact that the array is **sorted** is the massive clue that makes the optimal logic work.

# Solution 

## Brute Force (Nested Loops / Shifting)

- **Intuition:** A brute force way without extra memory would be to use a loop to count occurrences of the current number. If the count reaches 3, we use a second inner loop to physically shift every single element in the rest of the array one position to the left to overwrite the duplicate. 
- **Time Complexity:** $O(n^2)$ because shifting elements inside a loop is extremely slow.
- **Space Complexity:** $O(1)$.

## Optimized Code (Two Pointers)

- **Intuition:** Because the array is **sorted**, all identical numbers are grouped together.
  We can use two pointers:
  - `i` : The fast pointer that scans every element in the original array.
  - `k` : The slow pointer that tells us *where* to place the next valid number.
  
  **The Core Logic:** How do we know if `nums[i]` is a 3rd duplicate? 
  Since we are allowed to keep 2 duplicates, we compare the current number `nums[i]` with the number we placed 2 steps ago at `nums[k-2]`. 
  - If `nums[i] == nums[k-2]`, it means we already placed this exact number twice. Skip it!
  - If `nums[i] != nums[k-2]`, it is safe to keep! We place it at `nums[k]` and increment `k`.
- **Pseudo code:**
```text
1. IF nums.length <= 2: RETURN nums.length

2. SET k = 2 // Start placing new valid numbers at index 2 (indices 0 and 1 are always safe!)

3. FOR i from 2 to nums.length - 1:
4.     // Check if the current number is different from the one placed 2 spots ago
5.     IF nums[i] != nums[k-2]:
6.         nums[k] = nums[i]
7.         k++
8.         
9. RETURN k
```
- **Time Complexity:** $O(n)$. We only loop through the array exactly once.
- **Space Complexity:** $O(1)$. 
- **Solution Code (Java):**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // Edge case: if length is 2 or less, all elements are valid
        if (nums.length <= 2) {
            return nums.length;
        }
        
        // k is the pointer for where to place the next valid number.
        // We start at 2 because index 0 and 1 are always valid (we can have at most 2 duplicates)
        int k = 2; 
        
        for (int i = 2; i < nums.length; i++) {
            // Check against the newly built section of the array (k - 2)
            // If they match, it's a 3rd duplicate. If they don't, it's safe!
            if (nums[i] != nums[k - 2]) {
                nums[k] = nums[i];
                k++;
            }
        }
        
        return k;
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

1. **Thinking about HashMaps:**
   - *My mistake:* When a problem asks to count occurrences (like "at most twice"), the brain instantly wants to use a `HashMap`.
   - *Correction:* The constraints explicitly say `$O(1)$ extra memory`. HashMaps take $O(n)$ memory. Always read constraints carefully before jumping to HashMaps!

2. **Comparing `nums[i]` with `nums[i-2]` vs `nums[k-2]`:**
   - *My mistake:* The logic is to check if `nums[i] == nums[i-2]`.
   - *Correction:* While `nums[i-2]` makes conceptual sense, in code you must compare it against **`nums[k-2]`**! 
   Why? Because `k` represents the clean, newly built array we are constructing on the left side. `i` is just flying through the dirty original array. We want to check our current number against the *clean* array we've built, not the dirty one we are scanning!
