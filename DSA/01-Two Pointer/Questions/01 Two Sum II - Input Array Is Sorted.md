# 167. Two Sum II - Input Array Is Sorted

Given a **1-indexed** array of integers `numbers` that is already **sorted in non-decreasing order**, find two numbers such that they add up to a specific `target` number. Let these two numbers be `numbers[index1]` and `numbers[index2]` where `1 <= index1 < index2 <= numbers.length`.

Return *the indices of the two numbers, `index1` and `index2`, **each incremented by one**, as an integer array `[index1, index2]` of length 2.*

The tests are generated such that there is **exactly one solution**. You **may not** use the same element twice.

Your solution must use only constant extra space.

### Example 1:
**Input:** numbers = [2,7,11,15], target = 9
**Output:** [1,2]
**Explanation:** The sum of 2 and 7 is 9. Therefore, index1 = 1, index2 = 2. We return [1, 2].

### Constraints:
- `2 <= numbers.length <= 3 * 10^4`
- `-1000 <= numbers[i] <= 1000`
- `numbers` is sorted in **non-decreasing order**.
- `-1000 <= target <= 1000`
- The tests are generated such that there is **exactly one solution**.

---

# Understanding the Question 

Breaking down the problem before I jump into code:
- **Draw examples:** If `numbers = [2, 3, 4]` and `target = 6`, the answer is `[1, 3]` (since 2+4=6 and indices are 1-based, not 0-based).
- **Clarify edge cases:** The problem guarantees exactly one solution, and I can't reuse the same element (meaning my pointers can never be at the exact same index). 
- **Confirm input/output:** 
  - Input: A sorted array and a target int. 
  - Output: A 2-element array containing the 1-based indices.
- **Important keywords:** "sorted in non-decreasing order" (this is a huge hint!) and "constant extra space".
- **Basic understanding:** The goal is pretty simple—find two numbers summing to the target. But since the array is 1-indexed and sorted, I just need to return `[index1, index2]`. I can break out of the loop as soon as I find the exact pair.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n)$ or $O(n \log n)$ max.
- **Space complexity:** Strictly $O(1)$ space ("constant extra space").
- **Input/Output space:** The array can have up to 30,000 items. Output is always just 2 items.
- **What algorithm to use:** Two pointers seems perfect here.
- **How constraints guide the solution:** Since `numbers.length <= 30,000`, an $O(n^2)$ brute force algorithm would do roughly $900,000,000$ operations. A standard judge server does about $10^8$ ops/sec, so $O(n^2)$ will definitely hit a "Time Limit Exceeded" (TLE). I need something faster. Also, the strict $O(1)$ space rule completely kills the idea of using a Hash Map.

# Solution 

## Brute Force 

- **Intuition:** The dumbest way to do this is with nested loops. Just pick one number, then loop through the rest of the array to see if any subsequent number adds up to the target.
- **Pseudo code:**
```text
FOR i from 0 to array.length - 1:
    FOR j from i + 1 to array.length - 1:
        IF array[i] + array[j] == target:
            RETURN [i + 1, j + 1]
```
- **Time Complexity:** $O(n^2)$ because of the nested loops. (Will get TLE).
- **Space Complexity:** $O(1)$ since I'm not using any extra space.

## Better Approach (Hash Map)

- **Intuition:** Normally for Two Sum, I'd throw a Hash Map at it to store elements and their indices. That gets the time down to $O(n)$.
- **Why I can't use it:** A Hash Map requires $O(n)$ space complexity. The problem strictly says: *"Your solution must use only constant extra space."* So, this approach is out. Bummer.

## Optimized Code 

- **Intuition:** Okay, I have to use the fact that the array is **sorted**. I can just place two pointers: one at the start (smallest number) and one at the end (largest number). If their sum is too big, I move the right pointer to the left to get a smaller number. If the sum is too small, I move the left pointer to the right to get a bigger number. Simple!
- **Pseudo code:**
```text
1. SET left_pointer = 0
2. SET right_pointer = length of array - 1

3. WHILE left_pointer < right_pointer:
4.     CALCULATE current_sum = array[left_pointer] + array[right_pointer]
5.     IF current_sum == target:
6.         RETURN [left_pointer + 1, right_pointer + 1]
7.     ELSE IF current_sum < target:
8.         INCREMENT left_pointer by 1
9.     ELSE:
10.        DECREMENT right_pointer by 1
```
- **Time Complexity:** $O(n)$ because I only traverse the array once from the outside in.
- **Space Complexity:** $O(1)$ since I'm just using two integer variables (pointers).
- **Solution Code (Java):**
```java
class Solution {
    public int[] twoSum(int[] numbers, int target) { 
        int left = 0; 
        int right = numbers.length - 1;

        while (left < right) {
            int sum = numbers[left] + numbers[right];

            if (sum == target) {
                return new int[]{left + 1, right + 1}; 
            }
            else if (sum < target) {
                left++; 
            }
            else {
                right--; 
            }
        }
        return new int[] {-1, 1}; 
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

When I was first translating my pseudo-code into actual Java, I made a bunch of dumb syntax mistakes. Documenting them here so I don't repeat them in an actual interview:

1. **Method Signature**: 
   - *My mistake:* `public int [] twoSum(int [] number, int target []){` (Accidentally declared `target` as an array).
   - *Correction:* It needs to be a single integer: `int target`.
2. **Missing Semicolons**: 
   - *My mistake:* Kept forgetting to put `;` at the end of statements like `int left = 0` and `end --`.
   - *Correction:* It's Java, bro. Every statement needs a `;`.
3. **Variable Names Mix-up**: 
   - *My mistake:* I defined `left` and `right`, but then in the `if/else` block my brain switched to using `start++` and `end--`.
   - *Correction:* Gotta stay consistent! Stick to `left` and `right` everywhere.
4. **Returning an Array Syntax**: 
   - *My mistake:* `return new int[](start + 1, end + 1)'` 
   - *Correction:* To create a new array on the fly, use curly braces `{}`, not parentheses `()`. Also, watch out for random typos like that quote at the end. Correct syntax is `return new int[]{left + 1, right + 1};`.
5. **Brace Placement (Scope)**: 
   - *My mistake:* I put an extra closing brace `}` right before my final return statement. This kicked `return new int[] {-1, 1};` completely outside the method, causing a compiler error.
   - *Correction:* Make sure the final return statement stays inside the method body!