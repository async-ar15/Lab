# 387. First Unique Character in a String

Given a string `s`, find the **first non-repeating character** in it and return its index. If it does not exist, return `-1`.

### Example 1:
**Input:** s = "leetcode"
**Output:** 0
**Explanation:** The character 'l' at index 0 is the first character that does not occur at any other index.

### Example 2:
**Input:** s = "loveleetcode"
**Output:** 2

### Example 3:
**Input:** s = "aabb"
**Output:** -1

### Constraints:
- `1 <= s.length <= 10^5`
- `s` consists of only lowercase English letters.

---

# Understanding the Question 

Breaking down the problem before I jump into code:
- **Draw examples:** If `s = "aab"`, 'a' appears twice, 'b' appears once. The first unique is 'b' at index 2. Return `2`.
- **Clarify edge cases:** If every character repeats (like Example 3: `"aabb"`), there are no unique characters, so return `-1`.
- **Confirm input/output:**
  - Input: A single string `s`.
  - Output: An integer representing the index of the first non-repeating character, or `-1`.
- **Important keywords:** "first non-repeating" means we need to know the total counts of every character to verify if it's unique.
- **Basic understanding:** The goal is to find the first character that has a frequency of exactly 1. 

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n)$ or $O(n \log n)$ max.
- **Space complexity:** Usually $O(n)$ for hashing, but we can do $O(1)$.
- **Input/Output space:** The string can be up to 100,000 characters long.
- **What algorithm to use:** Hashing / Frequency counting.
- **How constraints guide the solution:** Since `s.length <= 10^5`, a brute force $O(n^2)$ approach will definitely hit a "Time Limit Exceeded" (TLE). We need something faster. Also, the constraint `s consists of only lowercase English letters` is a massive hint: there are exactly 26 lowercase English letters. Instead of using a heavy, dynamic `HashMap`, we can use a simple array of size 26 to keep track of character frequencies.

# Solution 

## Brute Force 

- **Intuition:** The dumbest way to do this is with nested loops. For every character in the string, scan the entire string again to see if that character appears anywhere else.
- **Pseudo code:**
```text
FOR i from 0 to string.length - 1:
    SET isUnique = true
    FOR j from 0 to string.length - 1:
        IF i != j AND string[i] == string[j]:
            isUnique = false
            BREAK
    IF isUnique == true:
        RETURN i
RETURN -1
```
- **Time Complexity:** $O(n^2)$ because for every character, we potentially scan the rest of the string. (Will get TLE).
- **Space Complexity:** $O(1)$ since we are just using loop variables.

## Better Approach (Hash Map)

- **Intuition:** Instead of checking the whole string for every character, we can use a Hash Map to count frequencies in one pass, and then find the first unique in a second pass.
- **Why we can optimize it further:** A Java `HashMap<Character, Integer>` has overhead (object creation, auto-boxing). Since the input is strictly limited to 26 lowercase letters, an array is much more efficient.

## Optimized Code (Array Map)

- **Intuition:** We use a two-pass approach with a primitive integer array of size 26.
  - **Pass 1:** Iterate through the string and count frequencies into `count[char - 'a']`.
  - **Pass 2:** Iterate through the string again from left to right. Check the count of the current character. The first one with a count of exactly `1` is our answer!
- **Pseudo code:**
```text
1. SET count = array of 26 integers (all initialized to 0)
2. FOR char c in string:
3.     INCREMENT count[c - 'a']
4. FOR i from 0 to string.length - 1:
5.     IF count[string[i] - 'a'] == 1:
6.         RETURN i
7. RETURN -1
```
- **Time Complexity:** $O(n)$ because we loop through the string exactly twice. Operations inside the loops are $O(1)$.
- **Space Complexity:** $O(1)$ because the `count` array is always exactly size 26, regardless of how large the string gets.
- **Solution Code (Java):**
```java
class Solution {
    public int firstUniqChar(String s) {
        int[] count = new int[26];
        
        // Pass 1: Build frequency map
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a']++;
        }
        
        // Pass 2: Find the first unique character
        for (int i = 0; i < s.length(); i++) {
            if (count[s.charAt(i) - 'a'] == 1) {
                return i;
            }
        }
        
        return -1;
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

When I was first trying to implement the Hash Map approach, I made a bunch of syntax and logic mistakes. Documenting them here so I don't repeat them:

1. **Nested Loops vs Sequential Loops:** 
   - *My mistake:* I put my second `for` loop *inside* the first `for-each` loop. 
   - *Correction:* A two-pass approach means the loops must be sequential. One loop must finish completely before the next one starts, otherwise it becomes $O(n^2)$ again!
2. **Populating the Map:** 
   - *My mistake:* In my first loop (typo `fro`), I never actually used `.put()` to add elements to the `map`. I just looped without doing anything.
   - *Correction:* Always remember to explicitly populate the map with `map.put(ch, map.getOrDefault(ch, 0) + 1)` to count the frequencies.
3. **Variable Redeclaration:** 
   - *My mistake:* I declared `char ch` in the first loop, and then declared `char ch = s.charAt(i)` again inside the inner loop. 
   - *Correction:* Variable names must be unique within their scope.
4. **Return Value:** 
   - *My mistake:* The method expects an `int` (the index), but I wrote `return;` inside the `if` statement.
   - *Correction:* Always match the return type! It needs to be `return i;`.