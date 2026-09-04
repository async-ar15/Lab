# 904. Fruit Into Baskets

Difficulty: Medium

You are visiting a farm that has a single row of fruit trees arranged from left to right. The trees are represented by an integer array `fruits` where `fruits[i]` is the **type** of fruit the $i^{th}$ tree produces.

You want to collect as much fruit as possible. However, the owner has some strict rules that you must follow:
- You only have **two baskets**, and each basket can only hold a **single type** of fruit. There is no limit on the amount of fruit each basket can hold.
- Starting from any tree of your choice, you must pick **exactly one fruit** from **every tree** (including the start tree) while moving to the right. The picked fruits must fit in one of your baskets.
- Once you reach a tree with fruit that cannot fit in your baskets, you must stop.

Given the integer array `fruits`, return the **maximum** number of fruits you can pick.

### Examples:

**Example 1:**
```
Input: fruits = [1,2,1]
Output: 3
Explanation: We can pick from all 3 trees.
```

**Example 2:**
```
Input: fruits = [0,1,2,2]
Output: 3
Explanation: We can pick from trees [1,2,2].
If we had started at the first tree, we would only pick from trees [0,1].
```

**Example 3:**
```
Input: fruits = [1,2,3,2,2]
Output: 4
Explanation: We can pick from trees [2,3,2,2].
If we had started at the first tree, we would only pick from trees [1,2].
```

### Constraints:
- `1 <= fruits.length <= 10^5`
- `0 <= fruits[i] < fruits.length`

---

# Understanding the Question

- **Input**: An integer array `fruits`.
- **Output**: The maximum number of fruits you can pick.
- **Simplification**: This problem boils down to finding the **longest contiguous subarray with at most 2 distinct integers**. (The 2 distinct integers represent the 2 baskets).

# Understanding the Constraints

- `fruits.length <= 10^5`: This constraint means an $O(N^2)$ brute force solution will give a Time Limit Exceeded (TLE). We must find an $O(N)$ or $O(N \log N)$ approach.
- `0 <= fruits[i] < fruits.length`: The values represent different types of fruits. We can use a `HashMap` to store fruit frequencies. Since we are restricted to at most 2 types, the HashMap size will never exceed 3, meaning the space complexity will be $O(1)$.

# Solution 

## 1. Brute Force 

- **Intuition**: Check every possible subarray. Use an outer loop `i` as the starting tree and an inner loop `j` to expand to the right. Keep adding fruits to a HashSet/HashMap. Stop the inner loop as soon as you see a 3rd distinct fruit (set size > 2), and record the maximum length.
- **Time Complexity**: $O(N^2)$ because of the nested loops. This will cause a TLE.
- **Space Complexity**: $O(1)$ because the Set/Map will never store more than 3 elements.

## 2. Optimized Approach (Sliding Window)

- **Intuition**: We use a variable-length sliding window with two pointers (`left` and `right`). 
  - **Expand**: We move `right` and add the current fruit to a frequency `HashMap`.
  - **Invalid Condition**: If the `HashMap` size exceeds 2, it means we have 3 types of fruits (our baskets are full). 
  - **Shrink**: We must shrink the window by moving `left` forward. We decrease the frequency of `fruits[left]` in our map. If its frequency drops to 0, we remove it from the map completely. We do this until the map size is back to 2.
  - **Update**: Record the maximum window length (`right - left + 1`).
- **Time Complexity**: $O(N)$. Both `left` and `right` pointers traverse the array at most once.
- **Space Complexity**: $O(1)$. The HashMap will never contain more than 3 entries.

### Code

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

# Mistakes & Corrections during the session

1. **Pseudo-code Logic Gap**: 
   - *Mistake*: In the pseudo-code, the logic fetched the current count of `fruits[right]` but forgot to actually update (put) the new count back into the map *before* checking the `while` loop condition.
   - *Correction*: Always make sure to fully add the new element to your tracking data structure (increment its frequency) before checking if your window has become invalid.
2. **Syntax Mixing**: 
   - *Mistake*: The pseudo-code mixed Java-like syntax with random curly braces closing prematurely (e.g., closing the method before the `for` loop started).
   - *Correction*: Keep structure clean, even in pseudo-code.