# Sliding Window — Revision Sheet

---

## 01. Max Sum Subarray of size K

**In My Words:** Given an array and integer K, find the maximum sum among all contiguous subarrays of exactly size K.

**Constraint Whispers:** 
- N ≤ 10⁶ → O(N²) will TLE (10¹² operations). Must be O(N).
- Max N is 10⁶ and Max elements are 10⁶. Summing them can result in 10¹². This exceeds the standard 32-bit `int` limit (~2 * 10⁹). We must use a `long` to prevent silent integer overflow!

**Brute Force:** Calculate the sum for every possible starting index using a nested loop. O(N * K) time.

**Why It Hurts:** Recalculates the overlapping middle elements over and over again. N*K operations cause a TLE.

**The Bridge:** When you shift a window of size K to the right by one step, K-1 elements stay exactly the same! The only thing that changes is that ONE old element falls out of the left side, and ONE new element enters from the right. We can calculate the new sum in O(1) time instead of O(K) time.

**Optimized Intuition:** Calculate the sum of the first `k` elements. This is our first window. Then, slide the window to the right: `currentSum = currentSum + newElement - oldElement`. Keep track of the maximum sum seen.

**Template:** Fixed-Size Sliding Window

**Gotcha:** (1) Remember to use `long` for the sum variables! (2) The formula for sliding is `+ arr[i] - arr[i - k]`. Be careful with indices.

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
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

---

## 02. Minimum Size Subarray Sum

**In My Words:** Given an array of positive integers and a target sum, find the minimum length of a contiguous subarray whose sum is >= target. Return 0 if impossible.

**Constraint Whispers:** 
- N ≤ 10⁵ → O(N²) will TLE. Must be O(N) or O(N log N).
- All numbers are POSITIVE. This guarantees monotonicity (adding expands sum, removing shrinks sum). This is the absolute requirement for a sliding window.

**Brute Force:** Try every starting index and keep adding elements until the sum >= target. Record the minimum length. O(N²) time.

**Why It Hurts:** Re-computes sums repeatedly. TLE.

**The Bridge:** Because all numbers are positive, if we hit the target, expanding the window further is pointless (we want the MINIMUM length). Instead, once we hit the target, we should try to *shrink* the window from the left to see if we can get an even smaller valid window! 

**Optimized Intuition:** Keep expanding `right` and adding to `sum`. `while (sum >= target)`, record the window length (`right - left + 1`), subtract `nums[left]`, and move `left` forward to shrink the window.

**Template:** Variable-Size Sliding Window

**Gotcha:** (1) You MUST use a `while (sum >= target)` loop to shrink, not an `if` statement! A single large number might allow you to shrink the left side multiple times. (2) Initialize `minLength` to `Integer.MAX_VALUE`. (3) At the end, return `minLength == Integer.MAX_VALUE ? 0 : minLength;` to handle the "impossible" case.

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        // Initialize to MAX_VALUE so our Math.min check works correctly
        int minLength = Integer.MAX_VALUE;
        int sum = 0;
        int left = 0;
        
        for (int right = 0; right < nums.length; right++) {
            sum += nums[right]; // Expand window by adding the right element
            
            // As long as the window meets the condition, try to shrink it from the left
            while (sum >= target) {
                // Record the current window's length
                minLength = Math.min(minLength, right - left + 1);
                
                // Shrink the window
                sum -= nums[left];
                left++;
            }
        }
        
        // If minLength never changed, we never hit the target, so return 0
        return minLength == Integer.MAX_VALUE ? 0 : minLength;
    }
}
```

---

## 03. Longest Substring with K Uniques

**In My Words:** Find the length of the longest substring that contains exactly K distinct characters. Return -1 if impossible.

**Constraint Whispers:** 
- N ≤ 10⁵ → O(N²) will TLE. Must be O(N).
- Characters are lowercase English letters. This means we don't need a heavy `HashMap`; we can use a fast, lightweight `int[26]` frequency array for O(1) space and hyper-fast lookups.

**Brute Force:** Generate all possible substrings, count unique characters in each, and track the max length for those with exactly K uniques. O(N²) time.

**Why It Hurts:** Generating all substrings takes too long (TLE). 

**The Bridge:** Instead of starting from scratch, if our window has > K unique characters, it is invalid. Because we want the longest substring, we expand the right side blindly. The moment we cross the limit (> K), we shrink from the left side *just enough* until we are back to exactly K uniques.

**Optimized Intuition:** Move `right` and update the frequency array. Keep a `uniqueCount`. `while (uniqueCount > K)`, shrink from the `left`. Decrease the frequency of the left character. If its frequency hits 0, `uniqueCount--`. If `uniqueCount == K`, update `maxLen`.

**Template:** Variable-Size Sliding Window (with Hash/Frequency Map)

**Gotcha:** (1) A `HashSet` is WRONG here! A HashSet only tells you *if* a char exists, not *how many*. When shrinking, you only drop a unique character when its frequency hits exactly 0. You must use a frequency map/array. (2) Initialize `maxLen = -1`.

**Time:** O(N) | **Space:** O(1) (fixed size 26 array)

**Code Solution:**
```java
class Solution {
    public int longestkSubstr(String s, int k) {
        int[] freq = new int[26];
        int left = 0;
        int maxLen = -1;
        int uniqueCount = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            
            // If it's a new character, increment our unique counter
            if (freq[rightChar - 'a'] == 0) {
                uniqueCount++;
            }
            freq[rightChar - 'a']++;
            
            // If unique characters exceed k, shrink the window from the left
            while (uniqueCount > k) {
                char leftChar = s.charAt(left);
                freq[leftChar - 'a']--;
                
                // If a character's frequency drops to 0, it's no longer in the window
                if (freq[leftChar - 'a'] == 0) {
                    uniqueCount--;
                }
                left++;
            }
            
            // If we have exactly k unique characters, update the max length
            if (uniqueCount == k) {
                maxLen = Math.max(maxLen, right - left + 1);
            }
        }
        
        return maxLen;
    }
}
```

---

## 04. Fruit Into Baskets

**In My Words:** Given an array of fruit types, find the longest contiguous sequence where you collect at most 2 different types of fruit.

**Constraint Whispers:** 
- N ≤ 10⁵ → O(N²) will TLE. Need O(N).
- Values are integers representing types. Since we can only hold 2 types, a HashMap tracking frequencies will never exceed size 3. Thus, space is O(1).

**Brute Force:** Try every starting tree, keep adding to a HashSet until you see a 3rd fruit type. Record max length. O(N²) time.

**Why It Hurts:** Re-evaluating the same trees over and over. TLE.

**The Bridge:** This problem is a cleverly disguised version of "Longest Substring with K Uniques" where K = 2! "Fruits" are characters, "baskets" are unique allowed characters. The logic is identical: expand right, if you get > 2 types, shrink left until you are back to 2 types.

**Optimized Intuition:** Move `right`, add fruit to a `HashMap` (FruitType -> Frequency). `while (map.size() > 2)`, shrink from `left`. Decrease the frequency of `fruits[left]`. If frequency hits 0, `map.remove(fruits[left])`. Update max length.

**Template:** Variable-Size Sliding Window (with HashMap)

**Gotcha:** (1) You MUST remove the key from the HashMap when its frequency drops to 0, otherwise `map.size()` will still count it! (2) The maximum length should be recorded *after* the while loop finishes shrinking, because at that point the window is guaranteed to be valid (<= 2 types).

**Time:** O(N) | **Space:** O(1) (Map size is at most 3)

**Code Solution:**
```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int totalFruit(int[] fruits) {
        Map<Integer, Integer> basket = new HashMap<>();
        int left = 0;
        int maxFruits = 0;
        
        for (int right = 0; right < fruits.length; right++) {
            // Add the current fruit to the basket
            basket.put(fruits[right], basket.getOrDefault(fruits[right], 0) + 1);
            
            // If we have more than 2 types of fruit, we need to shrink the window
            while (basket.size() > 2) {
                int leftFruit = fruits[left];
                basket.put(leftFruit, basket.get(leftFruit) - 1);
                
                // If the count of that fruit becomes 0, remove it from the basket entirely
                if (basket.get(leftFruit) == 0) {
                    basket.remove(leftFruit);
                }
                left++; // Shrink window from the left
            }
            
            // Update the maximum fruits we can collect
            maxFruits = Math.max(maxFruits, right - left + 1);
        }
        
        return maxFruits;
    }
}
```

---

# Pattern Recognition Cheat Sheet: Sliding Window

## The Universal Bridge Formula

**Brute Force → Optimal always follows this reasoning chain:**

1. **Brute force** = Nested loops generating every possible contiguous subarray/substring. (O(N²))
2. **Ask: "Am I recalculating overlapping parts?"**
   - In a subarray, shifting right by 1 means K-1 elements stay the same.
   - We can update the answer in O(1) by simply adding the new element and removing the old element.
3. **The Absolute Requirement:** The problem must involve **CONTIGUOUS** elements (subarray or substring).

## The Two Core Variants

| Variant | Purpose | Key Logic | Examples |
|---|---|---|---|
| **Fixed Size** | Find max/min/target in a window of exactly size K | Initialize first window. Slide by doing `sum += arr[i] - arr[i-k]`. | 01 (Max Sum K) |
| **Variable Size** | Find longest/shortest window that meets a condition | Expand `right` to add. `while (invalid)`, shrink `left` to remove. | 02, 03, 04 |

## The Golden Rules

1. **Monotonicity matters:** Sliding window only works if adding elements strictly moves you in one direction (e.g. sums only grow because numbers are positive). If there are negative numbers, the sum can fluctuate, breaking the logic (you'd need Prefix Sums instead).
2. **`while` for Shrinking:** In variable-size windows, ALWAYS use a `while` loop to shrink from the left, not an `if`. One bad element might require you to move `left` multiple times to fix the window.
3. **Frequencies over Sets:** When tracking unique elements in a window, a `HashSet` is dangerous because you don't know how many copies exist. Always use a Frequency Map/Array so you only remove a "unique" item when its count hits 0.
