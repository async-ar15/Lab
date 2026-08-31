# 26. Remove Duplicates from Sorted Array

Given an integer array `nums` sorted in **non-decreasing order**, remove the duplicates **in-place** such that each unique element appears only **once**. The **relative order** of the elements should be kept the **same**.

Consider the number of unique elements in `nums` to be `k`. After removing duplicates, return the number of unique elements `k`.

The first `k` elements of `nums` should contain the unique numbers in **sorted order**. The remaining elements beyond index `k - 1` can be ignored.

**Custom Judge:**
The judge will test your solution with the following code:
```java
int[] nums = [...]; // Input array
int[] expectedNums = [...]; // The expected answer with correct length

int k = removeDuplicates(nums); // Calls your implementation

assert k == expectedNums.length;
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}
```
If all assertions pass, then your solution will be **accepted**.

### Example 1:
**Input:** nums = [1,1,2]
**Output:** 2, nums = [1,2,_]
**Explanation:** Your function should return k = 2, with the first two elements of nums being 1 and 2 respectively.
It does not matter what you leave beyond the returned k (hence they are underscores).

### Example 2:
**Input:** nums = [0,0,1,1,1,2,2,3,3,4]
**Output:** 5, nums = [0,1,2,3,4,_,_,_,_,_]
**Explanation:** Your function should return k = 5, with the first five elements of nums being 0, 1, 2, 3, and 4 respectively.
It does not matter what you leave beyond the returned k (hence they are underscores).

### Constraints:
- `1 <= nums.length <= 3 * 10^4`
- `-100 <= nums[i] <= 100`
- `nums` is sorted in **non-decreasing order**.

---

# Understanding the Question 

Breaking down the problem before jumping into code:
- **Draw examples:** If `nums = [1, 1, 2]`, I need to change it to `[1, 2, _]` and return `2`. 
- **Clarify edge cases:** What if the array is just length 1? e.g., `[5]`. The answer is `1` and the array is untouched. The constraints say `nums.length >= 1`, so I don't have to worry about completely empty arrays.
- **Confirm input/output:** 
  - Input: An array of sorted integers `nums`.
  - Output: Return an integer `k` (the count of unique elements). But more importantly, the first `k` elements of `nums` must *be* those unique elements.
- **Important keywords:** "non-decreasing order" (sorted), "in-place" (no extra memory), "relative order ... kept the same".
- **Basic understanding:** I can't literally resize the array in Java/C++. "In-place" just means I need to shove all the unique numbers to the front of the array and return how many there are. The judge doesn't care what garbage data is left in the back half of the array.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** `nums.length <= 3 * 10^4`. This means an $O(n^2)$ algorithm will do $\approx 9 \times 10^8$ operations, which is dangerously close to a "Time Limit Exceeded" (TLE). I should aim for an $O(n)$ or $O(n \log n)$ approach.
- **Space complexity:** The "in-place" requirement means my space complexity must strictly be $O(1)$. 
- **What algorithm to use:** $O(n)$ time + $O(1)$ space on a sorted array = **Two Pointers**.

# Solution 

## Brute Force (Using Extra Space)

- **Intuition:** If the interviewer didn't force me to do it "in-place" in $O(1)$ space, the easiest way is just throwing a `HashSet` at it. Sets automatically remove duplicates.
- **Pseudo code:**
```text
SET seen = new HashSet()
FOR each num in nums:
    seen.add(num)

k = seen.size()
index = 0
FOR each uniqueNum in seen (in sorted order):
    nums[index] = uniqueNum
    index++

RETURN k
```
- **Time Complexity:** $O(n \log n)$ because adding to a TreeSet (to keep it sorted) takes time, or we'd have to sort it afterwards. (If we just iterate it would be $O(n)$ but a standard HashSet doesn't guarantee sorted order!).
- **Space Complexity:** $O(n)$ because we store up to $n$ unique elements in the Set.
- **Thoughts:** The space complexity breaks the rules of the question. We have to do it in $O(1)$ space!

## Optimized Code (Two Pointers)

- **Intuition:** Since the array is already sorted, duplicates are sitting right next to each other. We can just use two pointers. Let's call them `i` (the slow pointer) and `j` (the fast pointer).
  - `i` will keep track of where to place the *next* unique element we find. It will point to the *last unique element* we placed.
  - `j` will be our scout. It runs ahead in a loop scanning for new unique elements.
  - If `nums[i] == nums[j]`, `j` is just looking at a boring duplicate. It keeps moving.
  - If `nums[i] != nums[j]`, `j` found a brand new number! We step `i` forward by 1, and copy the new number from `nums[j]` into `nums[i]`.
  - At the end, since `i` is a 0-based index of where we put the last unique number, the total count of unique numbers is just `i + 1`.
- **Pseudo code:**
```text
1. SET i = 0
2. FOR j from 1 to length of nums - 1:
3.     IF nums[i] != nums[j]:
4.         i++
5.         nums[i] = nums[j] // Bring the unique number to the front!
6.
7. RETURN i + 1 // Count is index + 1
```
- **Time Complexity:** $O(n)$ because the `j` pointer just loops through the array exactly once.
- **Space Complexity:** $O(1)$ because we only use two integer variables (`i` and `j`).
- **Solution Code (Java):**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // Edge case: constraint says length >= 1, but always good practice
        if (nums.length == 0) return 0;

        int i = 0; // Tracks the index of the last unique element placed

        // j is our scout pointer scanning the array
        for (int j = 1; j < nums.length; j++) {
            // If we find a brand new unique element...
            if (nums[i] != nums[j]) {
                i++; // Move i forward
                nums[i] = nums[j]; // Place the new unique element at i
            }
            // If they are equal, j just naturally increments via the for-loop
        }

        // The number of unique elements is the index i + 1
        return i + 1;
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

Here's a record of things to watch out for if I code this in an interview:

1. **Forgetting to actually swap/assign the value:** 
   - *My mistake:* Writing `if (nums[i] != nums[j]) { i++; }`
   - *Correction:* Moving the pointer `i` isn't enough! You have to physically bring the unique number to the front of the array by adding `nums[i] = nums[j];` right after incrementing `i`.

2. **Returning the wrong count:**
   - *My mistake:* Returning `i` at the end of the method.
   - *Correction:* Arrays are 0-indexed. If `i` is pointing to index `3` (meaning we placed unique elements at `0, 1, 2, 3`), the total *count* of those elements is `4`. Always remember to return `i + 1`.

3. **Loop Bounds (Index out of bounds):**
   - *My mistake:* Doing a `while (j > n)` or confusing the length variables.
   - *Correction:* Standard `for` loops are safest here. `for (int j = 1; j < nums.length; j++)`. It clearly starts `j` at 1, goes until the end, and handles incrementing automatically.
