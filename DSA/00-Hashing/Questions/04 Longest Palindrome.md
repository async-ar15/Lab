# 409. Longest Palindrome

## Problem Statement
Given a string `s` which consists of lowercase or uppercase letters, return the length of the **longest palindrome** that can be built with those letters.

Letters are **case sensitive**, for example, `"Aa"` is not considered a palindrome.

## Examples
**Example 1:**
- Input: `s = "abccccdd"`
- Output: `7`
- Explanation: One longest palindrome that can be built is "dccaccd", whose length is 7.

**Example 2:**
- Input: `s = "a"`
- Output: `1`
- Explanation: The longest palindrome that can be built is "a", whose length is 1.

## Constraints
- `1 <= s.length <= 2000`
- `s` consists of lowercase **and/or** uppercase English letters only.

---

# Understanding the Question 

Breaking down the problem before I jump into code:
- **Draw examples:** If `s = "abbac"`, we can use two 'b's and two 'a's, and we can put one 'c' in the middle. The palindrome is "bacab", length 5.
- **Clarify edge cases:** "Aa" -> length 1 because case matters. "bb" -> length 2.
- **Confirm input/output:**
  - Input: A string `s` of characters.
  - Output: An integer representing the maximum possible length of a palindrome.
- **Important keywords:** "can be built with those letters". This means the original string doesn't have to be a palindrome, we can shuffle the letters around!
- **Basic understanding:** A palindrome is mirrored from the center. This means letters must come in *pairs* (evens). We can only ever have *one* letter sitting directly in the exact center that doesn't need a pair (meaning it appears an odd number of times). So, we need to pair up as many letters as possible, and if we have any leftovers, we can add exactly +1 to our final length.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n)$ where $n$ is `s.length`.
- **Space complexity:** $O(1)$ space because we only have a limited character set.
- **Input/Output space:** The string is relatively short (2000 chars), but an $O(n^2)$ algorithm might still be slow or over-complicated.
- **What algorithm to use:** Frequency counting / Hashing!
- **How constraints guide the solution:** `s` consists of lowercase **and/or** uppercase English letters. This means we can't just use `int[26]`. There are 52 letters in total (or we can just use an array of size 128 to cover all standard ASCII characters for safety and speed).

# Solution 

## Brute Force (Sorting)

- **Intuition:** We can sort the string's characters. Then we can scan through it and every time we see two identical characters next to each other, we increment our length by 2 and skip the next character.
- **Pseudo code:**
```text
CONVERT s to char array
SORT char array
SET length = 0
SET hasOdd = false
FOR i from 0 to array.length - 1:
    IF i < array.length - 1 AND array[i] == array[i+1]:
        length += 2
        i++ // Skip the paired character
    ELSE:
        hasOdd = true
IF hasOdd == true:
    length += 1
RETURN length
```
- **Time Complexity:** $O(N \log N)$ because sorting the string takes $N \log N$ time.
- **Space Complexity:** $O(N)$ or $O(\log N)$ depending on the sorting algorithm used to sort the character array.

## Optimized Code (Array Map)

- **Intuition:** We don't need to sort the string. We just need to know the *count* of each character. If a character appears 5 times, we can use 4 of them (which is `5 / 2 * 2`). We can do this for all characters. If we notice *any* character having an odd count, we know we can place one of those leftovers in the dead center, so we add 1 to the final length at the very end.
- **Why we optimize this way:** Since the string is only ASCII characters, we can use a fixed array of size 128 as our Hash Map. This gives us $O(N)$ time and $O(1)$ space.
- **Pseudo code:**
```text
1. SET count = array of 128 integers (all 0)
2. FOR char c in s:
3.     INCREMENT count[c]
4. SET length = 0
5. SET hasOdd = false
6. FOR i from 0 to 127:
7.     IF count[i] % 2 == 0:
8.         length += count[i]
9.     ELSE:
10.        length += count[i] - 1
11.        hasOdd = true
12. IF hasOdd:
13.    length += 1
14. RETURN length
```
- **Time Complexity:** $O(N)$ because we iterate through the string once, and then iterate through the fixed-size array of 128 elements once.
- **Space Complexity:** $O(1)$ because the integer array is always size 128 regardless of the input string length.
- **Solution Code (Java):**
```java
class Solution {
    public int longestPalindrome(String s) {
        int[] count = new int[128];
        
        // Pass 1: Build frequency map
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i)]++;
        }
        
        int length = 0;
        boolean hasOdd = false;
        
        // Pass 2: Calculate maximum pairs
        for (int i = 0; i < 128; i++) {
            if (count[i] % 2 == 0) {
                // If even, we can use all of them
                length += count[i];
            } else {
                // If odd, we can use the even portion (e.g. 5 -> use 4)
                length += count[i] - 1;
                hasOdd = true; // Flag that we have a leftover character
            }
        }
        
        // If we found any leftover character, we can put exactly one in the center
        if (hasOdd) {
            length += 1;
        }
        
        return length;
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

*(This section is currently empty since the file was directly populated without trial and error, but here are some common pitfalls for this problem):*

1. **Upper vs Lowercase Array Map:** 
   - *Mistake:* Using an array of size 26 and subtracting `'a'` like `count[c - 'a']`.
   - *Correction:* The constraints explicitly say it contains lowercase **and/or** uppercase English letters. 'A' (65) and 'a' (97) are far apart in ASCII. Using a size 128 array covers all standard ASCII cleanly without doing complex math.
2. **Adding too many odds:** 
   - *Mistake:* Trying to add `1` for *every* character that has an odd frequency.
   - *Correction:* You can only have exactly **one** character in the exact center of a palindrome. So you can only add `1` to the total length *once*, no matter how many odd-count letters you have leftover!
