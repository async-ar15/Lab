# 75. Sort Colors

You are given an array `nums` with `n` objects colored red, white, or blue, sort them **in-place** so that objects of the same color are adjacent, with the colors in the order red, white, and blue.

We will use the integers `0`, `1`, and `2` to represent the color red, white, and blue, respectively.

You must solve this problem without using the library's sort function.

### Example 1:
**Input:** nums = [2,0,2,1,1,0]
**Output:** [0,0,1,1,2,2]
**Explanation:** 
The array has two 0s, two 1s, and two 2s. Sorting them in-place places all 0s first, then all 1s, then all 2s.

### Example 2:
**Input:** nums = [2,0,1]
**Output:** [0,1,2]
**Explanation:** 
The array has one each of 0, 1, and 2, arranged in-place in the order 0, 1, 2.

### Constraints:
- `n == nums.length`
- `1 <= n <= 300`
- `nums[i]` is either `0`, `1`, or `2`.

**Follow up:** Could you come up with a one-pass algorithm using only constant extra space?

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** `[2,0,1]` needs to become `[0,1,2]`. `[1,1,0,2]` needs to become `[0,1,1,2]`.
- **Clarify edge cases:** The array might already be sorted, or it might only contain one color (e.g., all `1`s).
- **Confirm input/output:** 
  - Input: Array `nums`.
  - Output: Nothing (return type is `void`), we must modify the array **in-place**.
- **Important keywords:** "in-place", "one-pass algorithm", "constant extra space $O(1)$".
- **Basic understanding:** We cannot use `Arrays.sort()`. We cannot create a new array. We must shuffle the elements around inside the existing array until all `0`s are on the left, all `1`s are in the middle, and all `2`s are on the right.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** `nums.length <= 300`. This is incredibly small. A nested loop $O(n^2)$ algorithm like Bubble Sort or Selection Sort would only take about $90,000$ operations and would easily pass. However, the Follow-Up challenge strictly asks for a single pass ($O(n)$).
- **Space complexity:** Must be $O(1)$ constant space. We can only use a few integer variables.
- **What algorithm to use:** $O(n)$ Time + In-Place + Three Categories = **Dutch National Flag Algorithm (3 Pointers)**.

# Solution 

## Brute Force (Counting Sort - Two Passes)

- **Intuition:** Since we know the numbers can *only* be `0`, `1`, or `2`, the absolute easiest way to sort them is to just count how many of each we have! We loop through the array once to count them. Then we loop through the array a second time and overwrite it with the exact number of `0`s, then `1`s, then `2`s.
- **Pseudo code:**
```text
SET count0 = 0, count1 = 0, count2 = 0

FOR EACH num in nums:
    IF num == 0: count0++
    IF num == 1: count1++
    IF num == 2: count2++
    
FOR i from 0 to count0 - 1: nums[i] = 0
FOR i from count0 to count0 + count1 - 1: nums[i] = 1
FOR i from count0 + count1 to end: nums[i] = 2
```
- **Time Complexity:** $O(2n)$ which simplifies to $O(n)$, but it requires **two separate passes** over the array. 
- **Space Complexity:** $O(1)$ (just storing 3 count variables).

## Optimized Code (Dutch National Flag - One Pass)

- **Intuition:** We can sort this in exactly **one pass** using three pointers: `low`, `mid`, and `high`. 
  - `low` keeps track of the boundary for `0`s (everything before `low` is a `0`).
  - `high` keeps track of the boundary for `2`s (everything after `high` is a `2`).
  - `mid` is the "explorer". It scans the array from left to right.
  If `mid` finds a `0`, it tosses it to the left (swaps with `low`). If it finds a `2`, it tosses it to the right (swaps with `high`). If it finds a `1`, it leaves it alone and moves on!
- **Pseudo code:**
```text
1. SET low = 0, mid = 0, high = nums.length - 1

2. WHILE mid <= high:
3.     IF nums[mid] == 0:
4.         swap(nums[low], nums[mid])
5.         low++, mid++
6.     ELSE IF nums[mid] == 1:
7.         mid++
8.     ELSE IF nums[mid] == 2:
9.         swap(nums[mid], nums[high])
10.        high-- // DO NOT increment mid here!
```
- **Time Complexity:** $O(n)$. The `mid` pointer scans the array exactly once.
- **Space Complexity:** $O(1)$. 
- **Solution Code (Java):**
```java
class Solution {
    public void sortColors(int[] nums) {
        int low = 0;
        int mid = 0;
        int high = nums.length - 1;
        
        while (mid <= high) {
            if (nums[mid] == 0) {
                // Swap mid and low
                int temp = nums[low];
                nums[low] = nums[mid];
                nums[mid] = temp;
                
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                // Already in the right place
                mid++;
            } else if (nums[mid] == 2) {
                // Swap mid and high
                int temp = nums[high];
                nums[high] = nums[mid];
                nums[mid] = temp;
                
                high--;
                // We DO NOT increment mid here because the new number 
                // swapped from the high position needs to be checked!
            }
        }
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

1. **How to Swap in Java:**
   - *My mistake:* Writing `swap(nums[low], nums[mid])` exactly like the pseudo-code.
   - *Correction:* Java does not have a built-in `swap` function for array elements like C++ does. You have to manually swap the values using indices and a temporary variable (`temp`):
     ```java
     int temp = nums[i];
     nums[i] = nums[j];
     nums[j] = temp;
     ```

2. **The LeetCode Method Signature:**
   - *My mistake:* Trying to write `public static void main()`.
   - *Correction:* In LeetCode, you do not write the `main` method. The platform provides a specific method signature that you must use (e.g., `public void sortColors(int[] nums)`). The `nums` array is passed to you, and you just modify it inside that method!

3. **The "Don't Increment Mid" Trap:**
   - *My mistake:* It's very easy to accidentally put `mid++` inside the `nums[mid] == 2` block.
   - *Correction:* When you swap a number from the `high` end of the array, you have *no idea* what that number is yet. It could be a `0`, `1`, or `2`. Therefore, you must leave the `mid` pointer exactly where it is so it can evaluate that newly swapped number on the next iteration of the `while` loop!