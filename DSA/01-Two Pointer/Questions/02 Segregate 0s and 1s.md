# Segregate 0s and 1s

Given an array `arr[]` consisting of only `0`'s and `1`'s. Modify the array **in-place** to segregate `0`s onto the left side and `1`s onto the right side of the array.

### Examples:

**Input:** arr[] = [0, 1, 0, 1, 0, 0, 1, 1, 1, 0]
**Output:** [0, 0, 0, 0, 0, 1, 1, 1, 1, 1]
**Explanation:** After segregation, all the 0's are on the left and 1's are on the right. Modified array will be [0, 0, 0, 0, 0, 1, 1, 1, 1, 1].

**Input:** arr[] = [1, 1]
**Output:** [1, 1]
**Explanation:** There are no 0s in the given array, so the modified array is [1, 1]

### Constraints:
- `1 <= arr.size() <= 10^5`
- `0 <= arr[i] <= 1`

---

# Understanding the Question 

Breaking it down before writing any code:
- **Draw examples:** If `arr = [1, 0, 1, 0]`, the answer should be `[0, 0, 1, 1]`.
- **Clarify edge cases:** What if the array is already sorted? Like `[0, 0, 1, 1]`. It shouldn't break. What if it's all 0s or all 1s? Should still work fine without throwing an OutOfBounds exception. 
- **Confirm input/output:** 
  - Input: An array of 0s and 1s.
  - Output: The problem actually says "modify the array in-place", so usually I don't need to return anything (void method). The array just changes in memory.
- **Important keywords:** "in-place" is the big one. It screams $O(1)$ space complexity.
- **Basic understanding:** Move all the 0s to the left and all the 1s to the right. Literally just sorting, but specifically optimized for an array of binary values.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** Size is up to $10^5$. Pulling out the trusty cheat codes: $10^5$ means an $O(n^2)$ algorithm takes $10^{10}$ operations and will instantly get a "Time Limit Exceeded" (TLE). I have to do this in $O(n)$ or at worst $O(n \log n)$. 
- **Space complexity:** "In-place" explicitly locks me into $O(1)$ extra space. No creating new arrays to store the answer.
- **What algorithm to use:** $O(n)$ time + $O(1)$ space = Two Pointers. Perfect fit.

# Solution 

## Brute Force 

- **Intuition:** The dumbest/easiest way? Just call `Arrays.sort(arr)` (or whatever the language equivalent is). It will naturally put 0s before 1s. 
- **Pseudo code:**
```text
Sort(arr)
```
- **Time Complexity:** $O(n \log n)$ because of the sorting algorithm.
- **Space Complexity:** $O(1)$ to $O(\log n)$ depending on the internal sorting algorithm of the language.

## Better Approach (Counting)

- **Intuition:** Since we only have two possible values (0 and 1), we don't actually need to sort them like standard numbers. We can just count how many 0s there are in one pass. In a second pass, we just overwrite the beginning of the array with that many 0s, and fill the rest with 1s.
- **Pseudo code:**
```text
countZeroes = 0
FOR each num in arr:
    IF num == 0: countZeroes++

FOR i from 0 to array.length - 1:
    IF i < countZeroes:
        arr[i] = 0
    ELSE:
        arr[i] = 1
```
- **Time Complexity:** $O(n)$ since we do exactly two passes over the array ($N + N = 2N \rightarrow O(n)$).
- **Space Complexity:** $O(1)$ since we just use a single integer variable (`countZeroes`).
- **Thoughts:** This is actually pretty good and works perfectly for interviews, but the interviewer will usually hit you with a follow-up: *"Can you do it in exactly ONE pass?"*

## Optimized Code (Two Pointers)

- **Intuition:** To do it in one pass, we can place a `left` pointer at the start and a `right` pointer at the end. The `left` pointer looks for 1s (which are in the wrong place and need to move right). The `right` pointer looks for 0s (which need to move left). When they both find a misplaced number, we swap them!
- **Pseudo code:**
```text
1. SET left = 0
2. SET right = length of array - 1

3. WHILE left < right:
4.     // Move left pointer forward as long as it points to a 0
5.     WHILE arr[left] == 0 AND left < right:
6.         left++
7.     // Move right pointer backward as long as it points to a 1
8.     WHILE arr[right] == 1 AND left < right:
9.         right--
       
10.    // If they haven't crossed yet, it means left points to a 1 and right points to a 0
11.    IF left < right:
12.        SWAP arr[left] and arr[right]
13.        left++
14.        right--
```
- **Time Complexity:** $O(n)$ because the two pointers move towards each other and process each element exactly once. (One pass!).
- **Space Complexity:** $O(1)$ because we only use two integer pointers.
- **Solution Code (Java):**

```java
class Solution {
    public void segregate0and1(int[] arr) {
        int left = 0;
        int right = arr.length - 1;

        while (left < right) {
// Keep incrementing left index while we see 0 at left
            while (left < right && arr[left] == 0) {
                left++;
            }
// Keep decrementing right index while we see 1 at right
            while (left < right && arr[right] == 1) {
                right--;
            }

// If left < right, we found a 1 on the left and a 0 on the right, so swap them
            if (left < right) {
// Since we know the values are specifically 0 and 1, we can just hardcode the swap
                arr[left] = 0;
                arr[right] = 1;
                left++;
                right--;
            }
        }
    }
}
```
---

# Mistakes & Corrections

If I were to mess this up in an interview, here is where my bugs usually hide. Keeping a record of these to watch out for:

1. **Forgetting `left < right` inside the inner while loops**: 
   - *My mistake:* Writing `while (arr[left] == 0) { left++; }`
   - *Correction:* If the array is all 0s, the `left` pointer will just walk right off the edge of the array and throw an `ArrayIndexOutOfBoundsException`. I have to *always* check `left < right` before checking the array value to prevent it from going out of bounds.
2. **Swapping Values**:
   - *My mistake:* Writing a standard 3-line swap using a `temp` variable. `int temp = arr[left]; arr[left] = arr[right]; ...`
   - *Correction:* A standard swap works perfectly fine, but since we strictly know the values are *only* `0` and `1`, we can literally just hardcode `arr[left] = 0; arr[right] = 1;` after we find a mismatch. It's way cleaner and faster!
