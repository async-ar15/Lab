# 1189. Maximum Number of Balloons

## Problem Statement
Given a string `text`, you want to use the characters of `text` to form as many instances of the word **"balloon"** as possible.

You can use each character in `text` **at most once**. Return the maximum number of instances that can be formed.

## Examples
**Example 1:**
- Input: `text = "nlaebolko"`
- Output: `1`

**Example 2:**
- Input: `text = "loonbalxballpoon"`
- Output: `2`

**Example 3:**
- Input: `text = "leetcode"`
- Output: `0`

## Constraints
- `1 <= text.length <= 10^4`
- `text` consists of lower case English letters only.

---

# Understanding the Question 

Breaking down the problem before I jump into code:
- **Draw examples:** The word "balloon" has 7 letters: `b`, `a`, `l` (x2), `o` (x2), `n`. If I have `b:2, a:2, l:2, o:2, n:2`, I can only make **1** "balloon" because I need 4 `l`s and 4 `o`s to make 2.
- **Clarify edge cases:** If I'm missing even a single required letter (like `b`), I can't make any instances, so the answer is `0`.
- **Confirm input/output:**
  - Input: A string `text`.
  - Output: An integer representing how many times we can spell "balloon".
- **Important keywords:** "at most once". This means we just need to tally up our supply of letters. 
- **Basic understanding:** We just need to count how many `b`, `a`, `l`, `o`, and `n` characters we have. Since "balloon" needs two `l`s and two `o`s, we have to divide our supply of `l` and `o` by 2. The maximum number of words we can form is strictly bottlenecked by whichever required letter we have the *least* of.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $O(n)$ where $n$ is `text.length`.
- **Space complexity:** $O(1)$.
- **Input/Output space:** The string is up to 10,000 characters long.
- **What algorithm to use:** Frequency counting (Hashing / Array Map).
- **How constraints guide the solution:** `text` consists of lower case English letters only. Like our other hashing problems, this means we can use an `int[26]` array to count the frequencies instead of a full `HashMap`.

# Solution 

## Brute Force 

- **Intuition:** Try to repeatedly search for the letters 'b', 'a', 'l', 'l', 'o', 'o', 'n' in the string. If we find all of them, cross them out and increment our "words formed" counter. Keep doing this until we fail to find one of the letters.
- **Pseudo code:**
```text
CONVERT text to a mutable array or list of characters
SET balloons = 0
WHILE true:
    FOR char c IN "balloon":
        SET found = false
        FOR i from 0 to textArray.length - 1:
            IF textArray[i] == c:
                textArray[i] = '#' // mark as used
                found = true
                BREAK
        IF found == false:
            RETURN balloons
    balloons++
```
- **Time Complexity:** $O(N \times \frac{N}{7})$ which is effectively $O(N^2)$ in the worst case, because we repeatedly scan the remaining letters.
- **Space Complexity:** $O(N)$ to convert the string to a mutable array.

## Optimized Code (Array Map)

- **Intuition:** Instead of scanning multiple times, we scan `text` **once** to count the frequencies of all characters. We only care about `b`, `a`, `l`, `o`, and `n`. Then, we simply calculate how many full words we can make. We take the count of `b`, `a`, and `n`. For `l` and `o`, we take their count divided by 2 (integer division). The answer is the absolute minimum of these five values!
- **Pseudo code:**
```text
1. SET count = array of 26 integers (all 0)
2. FOR char c in text:
3.     INCREMENT count[c - 'a']
4. SET b = count['b' - 'a']
5. SET a = count['a' - 'a']
6. SET l = count['l' - 'a'] / 2
7. SET o = count['o' - 'a'] / 2
8. SET n = count['n' - 'a']
9. RETURN the minimum value among (b, a, l, o, n)
```
- **Time Complexity:** $O(N)$ because we iterate through the `text` string exactly once. Finding the minimum of 5 numbers takes $O(1)$ time.
- **Space Complexity:** $O(1)$ because our frequency array is always size 26.
- **Solution Code (Java):**
```java
class Solution {
    public int maxNumberOfBalloons(String text) {
        int[] count = new int[26];
        
        // Count frequencies of all characters
        for (int i = 0; i < text.length(); i++) {
            count[text.charAt(i) - 'a']++;
        }
        
        // Get the available instances for each required letter
        int b = count['b' - 'a'];
        int a = count['a' - 'a'];
        int l = count['l' - 'a'] / 2;
        int o = count['o' - 'a'] / 2;
        int n = count['n' - 'a'];
        
        // Find the limiting factor (the minimum)
        int min = b;
        min = Math.min(min, a);
        min = Math.min(min, l);
        min = Math.min(min, o);
        min = Math.min(min, n);
        
        return min;
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

1. **Forgetting to divide by 2:** 
   - *Mistake:* Just doing `Math.min` on the raw counts of `l` and `o`.
   - *Correction:* The word "balloon" requires **two** 'l's and **two** 'o's. So if you have 3 'l's, you can still only make 1 balloon. You MUST divide the counts of `l` and `o` by 2 (using integer division so `3 / 2 = 1`) before finding the minimum!
