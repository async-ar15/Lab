# Longest Substring with K Uniques

Difficulty: Medium

You are given a string `s` consisting only of lowercase alphabets and an integer `k`. Your task is to find the **length** of the **longest substring** that contains exactly `k` distinct characters.

**Note**: If no such substring exists, return `-1`.

### Examples:

**Example 1:**
```
Input: s = "aabacbebebe", k = 3
Output: 7
Explanation: The longest substring with exactly 3 distinct characters is "cbebebe", which includes 'c', 'b', and 'e'.
```

**Example 2:**
```
Input: s = "aaaa", k = 2
Output: -1
Explanation: There's no substring with 2 distinct characters.
```

**Example 3:**
```
Input: s = "aabaaab", k = 2
Output: 7
Explanation: The entire string "aabaaab" has exactly 2 unique characters 'a' and 'b', making it the longest valid substring.
```

### Constraints:
- 1 <= s.size() <= 10^5
- 1 <= k <= 26

---

# Understanding the Question

- **Input**: A string `s` of lowercase letters and an integer `k`.
- **Output**: An integer representing the max length of a substring with *exactly* `k` distinct characters, or `-1` if none exist.
- **Key Words**: "longest substring", "exactly k distinct". This strongly hints at a **Sliding Window** approach.
- **Edge Cases**:
  - The entire string has fewer than `k` distinct characters $\rightarrow$ return `-1`.
  - `s` is empty (though constraints say size >= 1).

# Understanding the Constraints

- $1 \le s.size() \le 10^5$: This is a massive hint! An $O(N^2)$ brute force will cause a Time Limit Exceeded (TLE). We strictly need an $O(N)$ or $O(N \log N)$ algorithm.
- $1 \le k \le 26$: Since there are only lowercase alphabets, we can use a custom array of size 26 instead of a HashMap to keep track of frequencies, giving us $O(1)$ space and extremely fast lookups.

# Solution 

## 1. Brute Force 

- **Intuition**: Generate all possible substrings. For each substring, count the number of unique characters. If it has exactly `k` unique characters, compare its length against the maximum length seen so far.
- **Time Complexity**: $O(N^2)$ to generate substrings, and checking uniqueness might push it to $O(N^3)$ or $O(N^2)$ depending on implementation. Will give TLE.
- **Space Complexity**: $O(1)$ or $O(K)$ to store character counts.

## 2. Optimized Approach (Sliding Window)

- **Intuition**: We use a variable-length sliding window with two pointers (`left` and `right`). As we expand the window by moving `right`, we track the frequencies of the characters in a size-26 array. If the number of unique characters exceeds `k`, we must shrink the window from the `left` until we are back to `k` unique characters. We update our `maxLen` whenever our unique character count is exactly `k`.
- **Time Complexity**: $O(N)$. Both `left` and `right` pointers traverse the string at most once. 
- **Space Complexity**: $O(1)$. We are using a fixed-size integer array `int[26]`, which takes constant space regardless of the input size.

### Code

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

# Mistakes & Corrections during the session

1. **HashSet vs Frequencies**: 
   - *Mistake*: Suggesting a `HashSet` to keep track of unique elements inside a sliding window.
   - *Correction*: A `HashSet` only tells you *if* an element exists, not *how many times* it exists. When shrinking a window, removing a character from a `HashSet` incorrectly assumes there are no other instances of that character left in the window. We must use a frequency map or an array.
2. **Data Structure Choice**:
   - *Mistake*: Being unsure why a custom array of size 26 is preferred over a `HashMap`.
   - *Correction*: Because the problem explicitly states "only lowercase alphabets," we have a fixed universe of 26 characters. An `int[26]` array is vastly faster and has lower overhead than a `HashMap<Character, Integer>`.
