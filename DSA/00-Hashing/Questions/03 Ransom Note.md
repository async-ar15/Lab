# 383. Ransom Note

Given two strings `ransomNote` and `magazine`, return `true` if `ransomNote` can be constructed by using the letters from `magazine` and `false` otherwise.

Each letter in `magazine` can only be used once in `ransomNote`.

### Example 1:
**Input:** ransomNote = "a", magazine = "b"
**Output:** false

### Example 2:
**Input:** ransomNote = "aa", magazine = "ab"
**Output:** false

### Example 3:
**Input:** ransomNote = "aa", magazine = "aab"
**Output:** true

### Constraints:
- `1 <= ransomNote.length, magazine.length <= 10^5`
- `ransomNote` and `magazine` consist of lowercase English letters.

---

# Understanding the Question 

Breaking down the problem before I jump into code:
- **Draw examples:** If `ransomNote = "abc"` and `magazine = "aabbcc"`, I have enough letters. Return `true`. If `ransomNote = "abc"` and `magazine = "ab"`, I am missing a 'c'. Return `false`.
- **Clarify edge cases:** If `ransomNote` is longer than `magazine`, it's impossible to construct it, so I can immediately return `false`.
- **Confirm input/output:**
  - Input: Two strings, `ransomNote` and `magazine`.
  - Output: A boolean (`true` or `false`).
- **Important keywords:** "can only be used once" means I have to keep track of the *count* of each letter available in the magazine.
- **Basic understanding:** We are given two strings. We need to check if we can generate string 1 (`ransomNote`) using the letters of string 2 (`magazine`). We are essentially checking if `magazine` has a superset of the characters (and their counts) needed for `ransomNote`.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n + m)$ where $n$ is `ransomNote.length` and $m$ is `magazine.length`.
- **Space complexity:** Usually $O(n)$ for hashing, but since it's just 26 letters, we can do $O(1)$.
- **Input/Output space:** Strings can be up to 100,000 characters long.
- **What algorithm to use:** Frequency counting / Hashing.
- **How constraints guide the solution:** Since lengths are up to $10^5$, an $O(n \times m)$ brute force approach (searching the magazine string from scratch for every letter in the ransom note and removing it) will hit a Time Limit Exceeded (TLE). Also, `consist of lowercase English letters` means we should use a size-26 integer array instead of a heavy `HashMap`.

# Solution 

## Brute Force 

- **Intuition:** For every character in `ransomNote`, scan `magazine` to find it. If found, mark it as "used" (e.g., replace it with a special character or use a visited array) and move to the next. If not found, return `false`.
- **Pseudo code:**
```text
IF ransomNote.length > magazine.length RETURN false

CONVERT magazine to an array of characters
FOR EACH char c IN ransomNote:
    SET found = false
    FOR i from 0 to magazine.length - 1:
        IF magazine[i] == c:
            magazine[i] = '#' // mark as used
            found = true
            BREAK
    IF found == false:
        RETURN false
RETURN true
```
- **Time Complexity:** $O(n \times m)$ where $n$ and $m$ are the lengths of the strings. For each character in the note, we might scan the entire magazine. (Will get TLE).
- **Space Complexity:** $O(m)$ to convert the magazine to a mutable array.

## Better Approach (Hash Map)

- **Intuition:** Instead of scanning the `magazine` repeatedly, scan it *once* and tally up the frequency of every available letter in a `HashMap<Character, Integer>`. Then, iterate through `ransomNote` and subtract from the tallies. If a tally drops below 0 (or a letter doesn't exist), return `false`.
- **Why we can optimize it further:** A Java `HashMap` has overhead. Since the input is strictly limited to 26 lowercase letters, an array map is much faster.

## Optimized Code (Array Map)

- **Intuition:** Using an array of 26 places, we first look at the `magazine` and count how many of each letter we have. Then we look at the `ransomNote` and subtract the letters we need.
  - **Pass 1:** Iterate through `magazine` and increment `count[char - 'a']`.
  - **Pass 2:** Iterate through `ransomNote` and decrement `count[char - 'a']`. If the count goes below 0, it means we don't have enough of that letter, so return `false`.
- **Pseudo code:**
```text
1. IF ransomNote.length > magazine.length RETURN false
2. SET count = array of 26 integers (all initialized to 0)

3. FOR char c in magazine:
4.     INCREMENT count[c - 'a']

5. FOR char c in ransomNote:
6.     DECREMENT count[c - 'a']
7.     IF count[c - 'a'] < 0:
8.         RETURN false

9. RETURN true
```
- **Time Complexity:** $O(m + n)$ because we loop through `magazine` once and `ransomNote` once.
- **Space Complexity:** $O(1)$ because the `count` array is always exactly size 26.
- **Solution Code (Java):**
```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        // Edge case optimization
        if (ransomNote.length() > magazine.length()) {
            return false;
        }
        
        int[] count = new int[26];
        
        // Count letters we have in the magazine
        for (int i = 0; i < magazine.length(); i++) {
            count[magazine.charAt(i) - 'a']++;
        }
        
        // Subtract letters we need for the ransom note
        for (int i = 0; i < ransomNote.length(); i++) {
            int index = ransomNote.charAt(i) - 'a';
            count[index]--;
            
            // If we went below 0, we didn't have enough of this letter
            if (count[index] < 0) {
                return false;
            }
        }
        
        return true;
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

1. **Forgetting the Length Edge Case:** 
   - A quick `if (ransomNote.length() > magazine.length()) return false;` at the very beginning skips unnecessary processing and speeds up the code significantly!
2. **Using a HashMap instead of an Array:** 
   - Not necessarily a "mistake", but it's much slower. Always use a size-26 `int[]` array when constraints specify "only lowercase English letters".
