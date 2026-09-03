# Max Sum Subarray of size K

Given an array of integers `arr[]` and a number `k`. Return the maximum sum of a subarray of size `k`.

**Note:** A subarray is a contiguous part of any given array.

### Example 1:
**Input:** arr[] = [100, 200, 300, 400], k = 2
**Output:** 700
**Explanation:** arr2 + arr3 = 700, which is maximum.

### Example 2:
**Input:** arr[] = [1, 4, 2, 10, 23, 3, 1, 0, 20], k = 4
**Output:** 39
**Explanation:** arr1 + arr2 + arr3 + arr4 = 39, which is maximum.

### Example 3:
**Input:** arr[] = [100, 200, 300, 400], k = 1
**Output:** 400
**Explanation:** arr3 = 400, which is maximum.

### Constraints:
- `arr.size() <= 10^6`
- `0 <= arr[i] <= 10^6`
- `1 <= k <= arr.size()`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** If `arr = [100, 200, 300, 400]` and `k = 2`. The subarrays of size 2 are:
  - `[100, 200]` -> sum = 300
  - `[200, 300]` -> sum = 500
  - `[300, 400]` -> sum = 700
  The maximum is 700.
- **Confirm input/output:** 
  - Input: Array `arr` and integer `k`.
  - Output: The maximum sum (should be a `long` due to constraints, see below).
- **Important keywords:** "contiguous", "subarray of size k". "Contiguous" immediately screams that we can use a **Sliding Window**.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** `arr.size() <= 10^6`. This means an $O(N^2)$ brute force solution will take $10^{12}$ operations and give a Time Limit Exceeded (TLE) error. We MUST solve this in $O(N)$.
- **The Sneaky Overflow Trap:** Look closely at the math. The max array size is $10^6$. The max value of a single element is $10^6$. If `k` is also $10^6$, the maximum possible sum is $10^6 \times 10^6 = 10^{12}$. 
  The maximum value a standard 32-bit `int` can hold is $\approx 2 \times 10^9$. Therefore, $10^{12}$ will overflow! We **must** use a 64-bit `long` to store our sum!

# Solution 

## Brute Force (Nested Loops)

- **Intuition:** The simplest way is to check every single possible subarray of size `k`. We lock our starting position `i`, and then use an inner loop `j` to add up the next `k` elements.
- **Pseudo code:**
```text
SET maxSum = 0

FOR i from 0 to arr.length - k:
    SET currentSum = 0
    FOR j from i to i + k - 1:
        currentSum = currentSum + arr[j]
        
    maxSum = Math.max(maxSum, currentSum)

RETURN maxSum
```
- **Time Complexity:** $O(N \cdot K)$. If `N` is $10^6$ and `K` is $10^5$, this will time out easily.
- **Space Complexity:** $O(1)$.

## Optimized Code (Fixed-Size Sliding Window)

- **Intuition:** Why are we recalculating the sum from scratch every single time? 
  Look at the shift from `[100, 200]` to `[200, 300]`.
  The only difference is that we dropped the `100` and picked up the `300`. The `200` stayed exactly the same!
  
  This is the core of the **Sliding Window** pattern:
  1. Calculate the sum of the very first window of size `k`.
  2. To slide the window to the right, just **add the new element coming in** and **subtract the old element falling out**.
- **Pseudo code:**
```text
SET maxSum = 0
SET currentWindowSum = 0

// 1. Create the first window
FOR i from 0 to k - 1:
    currentWindowSum = currentWindowSum + arr[i]

maxSum = currentWindowSum

// 2. Slide the window across the rest of the array
FOR i from k to arr.length - 1:
    // Add the new element, subtract the element that fell out of the window
    currentWindowSum = currentWindowSum + arr[i] - arr[i - k]
    maxSum = Math.max(maxSum, currentWindowSum)

RETURN maxSum
```
- **Time Complexity:** $O(N)$. We only pass through the array a single time!
- **Space Complexity:** $O(1)$. 

- **Solution Code (Java):**
```java
class Solution {
    public long maximumSumSubarray(int[] arr, int k) {
        long maxSum = 0;
        long currentWindowSum = 0;
        
        // Step 1: Calculate the sum of the very first window
        for (int i = 0; i < k; i++) {
            currentWindowSum += arr[i];
        }
        
        maxSum = currentWindowSum;
        
        // Step 2: Slide the window
        for (int i = k; i < arr.length; i++) {
            // arr[i] is the new element entering the window on the right
            // arr[i - k] is the old element leaving the window on the left
            currentWindowSum = currentWindowSum + arr[i] - arr[i - k];
            
            // Update the maximum sum
            maxSum = Math.max(maxSum, currentWindowSum);
        }
        
        return maxSum;
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

1. **Integer Overflow:**
   - *Common Mistake:* Using `int currentWindowSum` and `int maxSum`.
   - *Correction:* Always read the constraints! Maximum values of $10^6$ being added together up to $10^6$ times equals $10^{12}$. An `int` tops out at $\approx 2.14 \times 10^9$. Using `int` will cause silent overflow errors where your sums suddenly turn into negative numbers! Use `long`.

2. **Re-calculating the sum:**
   - *Common Mistake:* Using a nested loop to calculate the sum from scratch for every window.
   - *Correction:* The Sliding Window technique exists specifically to avoid this duplicate work. Just take your current sum, add the newly entering element, and subtract the newly exiting element. $O(1)$ math instead of an $O(K)$ loop!
