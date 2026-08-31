# 977. Squares of a Sorted Array

Given an integer array `nums` sorted in **non-decreasing** order, return *an array of **the squares of each number** sorted in non-decreasing order*.

### Example 1:
**Input:** nums = [-4,-1,0,3,10]
**Output:** [0,1,9,16,100]
**Explanation:** After squaring, the array becomes [16,1,0,9,100].
After sorting, it becomes [0,1,9,16,100].

### Example 2:
**Input:** nums = [-7,-3,2,3,11]
**Output:** [4,9,9,49,121]

### Constraints:
- `1 <= nums.length <= 10^4`
- `-10^4 <= nums[i] <= 10^4`
- `nums` is sorted in **non-decreasing** order.

**Follow up:** Squaring each element and sorting the new array is very trivial, could you find an `O(n)` solution using a different approach?

---

# Understanding the Question 

Breaking down the problem before jumping into code:
- **Draw examples:** If `nums = [-3, 0, 1, 2]`. The squares are `9, 0, 1, 4`. Sorted, that's `[0, 1, 4, 9]`.
- **Clarify edge cases:** The array could be entirely negative `[-5, -3, -1]` or entirely positive `[1, 2, 3]`. The length can be up to $10^4$, so we don't have to worry about empty arrays (`nums.length >= 1`).
- **Confirm input/output:** 
  - Input: An array of integers `nums` (can be negative and positive).
  - Output: A *new* array of the squares of those integers, sorted in increasing order.
- **Important keywords:** "sorted in non-decreasing order", "squares", "Follow up: O(n) solution".
- **Basic understanding:** The trick here is that negative numbers, when squared, become positive. A very large negative number (like `-100`) will have a huge square (`10000`), which completely ruins the sorted order if we just square the array in-place from left to right.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** The follow-up explicitly dares us to find an $O(n)$ solution. We must beat the trivial $O(n \log n)$ approach of just squaring and sorting.
- **Space complexity:** Since we are required to *return an array*, we must allocate a new array of size $N$. Therefore, our space complexity will be $O(n)$. We can't do this strictly in-place in $O(1)$ space without a massive headache.
- **What algorithm to use:** $O(n)$ time + sorted array usually means **Two Pointers**. Since the largest squares are at the opposite ends of the array, we should put one pointer at the start and one at the end!

# Solution 

## Brute Force (Square and Sort)

- **Intuition:** The easiest way to solve this is to just do exactly what the problem description says: square every number in the array, and then use the language's built-in sorting method to sort it.
- **Pseudo code:**
```text
FOR i from 0 to nums.length - 1:
    nums[i] = nums[i] * nums[i]
    
sort(nums)
RETURN nums
```
- **Time Complexity:** $O(n \log n)$ because of the sorting step. 
- **Space Complexity:** $O(1)$ auxiliary space (if we modify the input array in-place before sorting it), or $O(\log n) / O(n)$ depending on the language's sorting algorithm under the hood. 
- **Thoughts:** This is way too easy. The interviewer will definitely ask for the $O(n)$ follow-up solution.

## Optimized Code (Two Pointers from Outside-In)

- **Intuition:** Because the array is sorted, the absolute largest values (the ones that will produce the biggest squares) are always sitting at the extreme ends of the array (either very negative numbers on the far left, or very positive numbers on the far right). 
  We can use two pointers (`left` at `0`, and `right` at `n-1`). Whichever pointer has the larger square gets placed at the *end* of our new `result` array, and we move that pointer inward. We fill the `result` array from right-to-left (largest to smallest)!
- **Pseudo code:**
```text
1. n = length of nums
2. SET result = new array of size n
3. SET left = 0, right = n - 1
4. SET index = n - 1 // Start filling the result array from the end

5. WHILE left <= right:
6.     leftSquare = nums[left] * nums[left]
7.     rightSquare = nums[right] * nums[right]
8.     
9.     IF leftSquare > rightSquare:
10.        result[index] = leftSquare
11.        left++
12.    ELSE:
13.        result[index] = rightSquare
14.        right--
15.        
16.    index--

17. RETURN result
```
- **Time Complexity:** $O(n)$ because we process each element exactly once with the two pointers.
- **Space Complexity:** $O(n)$ because we create a new `result` array to store the answers.
- **Solution Code (Java):**
```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] result = new int[n]; // O(n) space for the answer
        
        int left = 0;
        int right = n - 1;
        int index = n - 1; // Start at the end of the new array
        
        while (left <= right) {
            int leftSquare = nums[left] * nums[left];
            int rightSquare = nums[right] * nums[right];
            
            // Compare the squares and pick the largest one
            if (leftSquare > rightSquare) {
                result[index] = leftSquare;
                left++; // Move the left pointer inward
            } else {
                result[index] = rightSquare;
                right--; // Move the right pointer inward
            }
            index--; // Move to the next empty spot in our result array
        }
        
        return result;
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

Things to remember for this specific problem:

1. **Trying to do it in-place ($O(1)$ space)**:
   - *My mistake:* Trying to overwrite the `nums` array while traversing it.
   - *Correction:* If you overwrite the back of the array with a large square, you destroy the number that was sitting there before you have a chance to square it! You *must* use a new array. Space complexity is $O(n)$.

2. **Loop Condition**:
   - *My mistake:* Writing `while (left < right)` instead of `while (left <= right)`.
   - *Correction:* If you use `<`, you will miss the very last element when `left` and `right` point to the exact same index! The loop must run while they are less than OR equal to each other so the last element gets squared and added to `result[0]`.

3. **Filling the array from left-to-right**:
   - *My mistake:* Trying to find the *smallest* square first.
   - *Correction:* Finding the smallest square first is a nightmare because you have to figure out where the negative numbers stop and the positive numbers begin (finding the "valley" of the array). It is infinitely easier to find the *largest* squares by starting at the extreme ends (`0` and `n-1`) and filling the result array backwards (`index = n - 1`).
