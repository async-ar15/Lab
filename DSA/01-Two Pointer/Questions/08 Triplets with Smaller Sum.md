# Triplets with Smaller Sum

Given an array `arr[]` of distinct integers and an integer `sum`, count the number of unique triplets of elements whose sum is strictly less than `sum`. A triplet is identified only by the three elements it contains, so different permutations of the same three elements are counted as one triplet.

### Example 1:
**Input:** sum = 2, arr[] = [-2, 0, 1, 3]
**Output:** 2
**Explanation:** Triplets with sum less than 2 are (-2, 0, 1) and (-2, 0, 3).

### Example 2:
**Input:** sum = 12, arr[] = [5, 1, 3, 4, 7]
**Output:** 4
**Explanation:** Triplets with sum less than 12 are (1, 3, 4), (5, 1, 3), (1, 3, 7) and (5, 1, 4).

### Constraints:
- `1 <= sum <= 10^5`
- `3 <= arr.size() <= 10^3`
- `-10^3 <= arr[i] <= 10^3`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** If `arr[] = [-2, 0, 1, 3]` and `sum = 2`. The combinations that are strictly less than 2 are `(-2) + 0 + 1 = -1` (valid) and `(-2) + 0 + 3 = 1` (valid). Total count is 2.
- **Clarify edge cases:** "Strictly less than sum" means `< sum`, not `<= sum`. We are returning a COUNT of triplets, not the triplets themselves.
- **Confirm input/output:** 
  - Input: An array `arr[]` and an integer `sum`.
  - Output: An integer representing the *count* of valid triplets.
- **Important keywords:** "count", "triplets", "strictly less than".
- **Basic understanding:** Again, this is a `3Sum` variant. We are looking for three numbers, so sorting the array and using a `for` loop + Two Pointers is the way to go. The difficult part is the "math"—how do we count efficiently without checking every single combination one by one?

# Understanding the Constraints

- **Time complexity:** `arr.size() <= 10^3` ($1000$). An $O(n^3)$ algorithm takes $10^9$ operations (Time Limit Exceeded). An $O(n^2)$ algorithm takes $1,000,000$ operations (Fast and passes easily). We need $O(n^2)$.
- **Space complexity:** We just need an integer to hold the `count`, so $O(1)$.
- **What algorithm to use:** $O(n^2)$ Time + Triplets = **Sorting + Two Pointers**.

# Solution 

## Brute Force (Three Loops)

- **Intuition:** Run 3 nested loops, sum up every triplet, and if the sum is less than the target, increment a counter.
- **Pseudo code:**
```text
SET count = 0
FOR i from 0 to n-1:
    FOR j from i+1 to n-1:
        FOR k from j+1 to n-1:
            IF arr[i] + arr[j] + arr[k] < sum:
                count++
RETURN count
```
- **Time Complexity:** $O(n^3)$. Will definitely TLE.
- **Space Complexity:** $O(1)$.

## Optimized Code (Sorting + Two Pointers)

- **Intuition:** We sort the array and lock in the first number `arr[i]`. Then we set `left` and `right` pointers. 
  
  **The Mathematics trick:** If `arr[i] + arr[left] + arr[right] < sum`, we found a valid triplet! But wait... because the array is sorted, if `arr[right]` is valid, then ANY element between `left` and `right` will ALSO be valid when paired with `left`. 
  For example, if `[..., 2, 3, 5, 8]` and `left` is on `2` and `right` is on `8`, and they form a valid sum... then `2` paired with `5` is valid, and `2` paired with `3` is valid.
  Instead of moving `right` down one by one and counting, we can just do the math: there are `(right - left)` valid pairs starting with `left`! We add that to our `count` instantly, and then move `left++` to look for new combinations.
- **Pseudo code:**
```text
1. Sort the arr
2. SET count = 0

3. FOR i from 0 to length - 1:
4.     SET left = i + 1
5.     SET right = length - 1
6.    
7.     WHILE left < right:
8.         SET currentSum = arr[i] + arr[left] + arr[right]
9.         
10.        IF currentSum < sum:
11.            // The Mathematics: Add ALL valid pairs instantly
12.            count = count + (right - left)
13.            left++ // Move left to find more valid triplets
14.        ELSE:
15.            // The sum is too big (or equal), we must shrink it
16.            right--

17. RETURN count
```
- **Time Complexity:** $O(n \log n)$ to sort + $O(n^2)$ for the loops = **$O(n^2)$** overall time.
- **Space Complexity:** $O(1)$ auxiliary space.
- **Solution Code (Java):**
```java
class Solution {
    long countTriplets(long arr[], int n, int sum) {
        Arrays.sort(arr);
        long count = 0;
        
        for (int i = 0; i < n; i++) {
            int left = i + 1;
            int right = n - 1;
            
            while (left < right) {
                long currentSum = arr[i] + arr[left] + arr[right];
                
                if (currentSum < sum) {
                    // The Mathematics trick:
                    // If arr[right] works with arr[left], then everything 
                    // before right will also work with arr[left] because it's sorted!
                    count += (right - left);
                    left++; // Now let's try a bigger left
                } else {
                    // Too big or equal, need to shrink the sum
                    right--;
                }
            }
        }
        
        return count;
    }
}
```

---

# Mistakes & Corrections

The main traps for this problem revolve around the mathematics:

1. **Forgetting to count `(right - left)`:**
   - *My mistake:* I just did `count++` and then `left++` when I found a sum that was smaller than the target.
   - *Correction:* This completely skips checking the other elements between `left` and `right`. If `left` and `right` work, then `left` and `right-1` definitely work, `left` and `right-2` work, etc. You MUST count all of them at once using `count += (right - left)`.

2. **Using the wrong loop condition:**
   - *My mistake:* Writing `if (currentSum <= sum)`.
   - *Correction:* The question explicitly asks for triplets whose sum is **strictly less than** `sum`. If it equals the sum, it is invalid, so you must treat it like it's too big and do `right--`.
