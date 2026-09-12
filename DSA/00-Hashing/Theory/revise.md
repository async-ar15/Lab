# Hashing — Revision Sheet

---

## 01. Two Sum

**In My Words:** Given an array of integers and a target sum, find the indices of the two numbers that add up to the target.

**Constraint Whispers:** 
- n ≤ 10⁴ → O(n²) works but is borderline slow. Interviewers will demand O(n).
- "Exactly one solution" → No need to worry about multiple pairs or returning empty arrays.
- Not sorted → Two Pointers won't work in O(n). Sorting it first would take O(n log n).

**Brute Force:** Nested loops checking every single pair combination. O(n²) time, O(1) space.

**Why It Hurts:** 10,000² = 100,000,000 operations. It's too slow and ignores the fact that we can trade space for time.

**The Bridge:** If I am at number `x`, I am looking for `target - x`. Instead of searching the rest of the array for it, what if I just remembered everything I've seen so far? If I drop every number into a HashMap as I go, checking if my complement exists becomes an instant $O(1)$ lookup.

**Optimized Intuition:** Create a HashMap (value -> index). Iterate through the array. For each number, calculate `complement = target - currentNumber`. Check if `complement` is in the map. If yes, we found the pair! If no, put the `currentNumber` and its index into the map.

**Template:** The Complement Search (Remembering)

**Gotcha:** You MUST check the map *before* adding the current number to it, otherwise you might use the same number twice if `target = 2 * currentNumber`.

**Time:** O(n) | **Space:** O(n)

**Code Solution:**
```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            if (map.containsKey(complement)) {
                return new int[] {map.get(complement), i};
            }
            
            map.put(nums[i], i);
        }
        
        return new int[] {};
    }
}
```

---

## 02. First Unique Character in a String

**In My Words:** Given a string, find the index of the first character that only appears exactly once. Return -1 if all characters repeat.

**Constraint Whispers:** 
- n ≤ 10⁵ → O(n²) will hit TLE (10¹⁰ ops). Must be O(n).
- "only lowercase English letters" → This is a huge hint to use an Array Map (`int[26]`) instead of a heavy `HashMap`.

**Brute Force:** Nested loops. For every character, scan the whole string again to see if it appears anywhere else. O(n²) time.

**Why It Hurts:** 10¹⁰ operations will fail. We need to stop scanning the string repeatedly.

**The Bridge:** To know if a character is unique, we must know the total count of every character in the string. If we count all frequencies in one pass, we can just do a second pass to find the first character whose count is exactly 1.

**Optimized Intuition:** Two passes. Pass 1: Build a frequency array of size 26. Pass 2: Iterate through the string from left to right. The first character we hit that has `count == 1` in our array is the answer.

**Template:** Frequency Map (using Array Map constraint optimization)

**Gotcha:** Loop through the *string* on the second pass, not the frequency array! Looping through the array will give you the first unique character alphabetically, not the first one as it appeared in the string.

**Time:** O(n) | **Space:** O(1) (Array is always size 26)

**Code Solution:**
```java
class Solution {
    public int firstUniqChar(String s) {
        int[] count = new int[26];
        
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a']++;
        }
        
        for (int i = 0; i < s.length(); i++) {
            if (count[s.charAt(i) - 'a'] == 1) {
                return i;
            }
        }
        
        return -1;
    }
}
```

---

## 03. Ransom Note

**In My Words:** Given a `ransomNote` string and a `magazine` string, check if you can spell the ransom note using the letters in the magazine. Each magazine letter can be used only once.

**Constraint Whispers:** 
- Lengths ≤ 10⁵ → O(n²) will hit TLE.
- "consist of lowercase English letters" → Array Map (`int[26]`) time!

**Brute Force:** For every letter in `ransomNote`, scan `magazine` to find it, cross it out if found, return false if not found. O(n * m) time.

**Why It Hurts:** Scanning the magazine repeatedly is too slow.

**The Bridge:** If we just tally up all the resources we have in the `magazine` first, we can simply check if we have enough of each resource to cover the `ransomNote`. It becomes a simple accounting problem.

**Optimized Intuition:** Create an `int[26]` frequency array. Pass 1: Iterate through `magazine` and add to the counts. Pass 2: Iterate through `ransomNote` and subtract from the counts. If any count ever drops below 0, we don't have enough letters, so return false.

**Template:** Frequency Map (Accounting / Validation)

**Gotcha:** Don't forget the $O(1)$ edge case: if `ransomNote.length() > magazine.length()`, it's physically impossible, return false immediately to save time.

**Time:** O(n + m) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        if (ransomNote.length() > magazine.length()) return false;
        
        int[] count = new int[26];
        
        for (int i = 0; i < magazine.length(); i++) {
            count[magazine.charAt(i) - 'a']++;
        }
        
        for (int i = 0; i < ransomNote.length(); i++) {
            int index = ransomNote.charAt(i) - 'a';
            count[index]--;
            if (count[index] < 0) return false;
        }
        
        return true;
    }
}
```

---

## 04. Longest Palindrome

**In My Words:** Given a string, shuffle the letters around to build the longest possible palindrome. Return that maximum length. (Case sensitive).

**Constraint Whispers:** 
- n ≤ 2000 → O(n²) would pass, but O(n) is expected.
- "lowercase and/or uppercase" → You cannot use an `int[26]` array! You need an `int[128]` array to cover both 'A' and 'a'.

**Brute Force:** Sort the characters, then count adjacent pairs. O(n log n).

**Why It Hurts:** Sorting is unnecessary overhead just to count pairs. 

**The Bridge:** A palindrome requires letters to come in pairs (evens). We can only have exactly ONE leftover (odd) letter sit in the very center. So, we just need the frequencies. If a letter appears 5 times, we can use 4 of them. If any odd-counted letters exist, we can add exactly +1 to our final length.

**Optimized Intuition:** Count frequencies using an `int[128]` array. Loop through the array. For any count, add `count` if it's even, or `count - 1` if it's odd. If we see any odd count, flip a `hasOdd` flag. At the end, if `hasOdd` is true, add 1 to the total length.

**Template:** Frequency Map (Pairing)

**Gotcha:** Remember that you can only add `+1` for an odd character **ONCE** per string, because a palindrome only has exactly one center spot!

**Time:** O(n) | **Space:** O(1) (Size 128 array)

**Code Solution:**
```java
class Solution {
    public int longestPalindrome(String s) {
        int[] count = new int[128];
        
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i)]++;
        }
        
        int length = 0;
        boolean hasOdd = false;
        
        for (int i = 0; i < 128; i++) {
            if (count[i] % 2 == 0) {
                length += count[i];
            } else {
                length += count[i] - 1;
                hasOdd = true;
            }
        }
        
        if (hasOdd) length++;
        
        return length;
    }
}
```

---

## 05. Maximum Number of Balloons

**In My Words:** Given a string, how many times can you form the word "balloon" using the characters? Characters can only be used once.

**Constraint Whispers:** 
- n ≤ 10⁴ → O(n) required.
- "lowercase English letters" → `int[26]` array map.

**Brute Force:** Repeatedly scan for the 7 letters and cross them out until you can't find one. O(n²).

**Why It Hurts:** Scanning over and over is too slow.

**The Bridge:** This is just a frequency counting problem. The limiting factor (bottleneck) determines the maximum number of words we can form. 

**Optimized Intuition:** Count frequencies in one pass. Then check our supplies of 'b', 'a', 'l', 'o', and 'n'. Since "balloon" needs TWO 'l's and TWO 'o's, we must divide their counts by 2. The final answer is just `Math.min()` across those 5 values.

**Template:** Frequency Map (Bottleneck / Ratio)

**Gotcha:** Forgetting to divide `l` and `o` by 2! Having 3 'l's only gives you enough for 1 balloon. `3 / 2 = 1` (integer division naturally handles this perfectly).

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int maxNumberOfBalloons(String text) {
        int[] count = new int[26];
        
        for (int i = 0; i < text.length(); i++) {
            count[text.charAt(i) - 'a']++;
        }
        
        int b = count['b' - 'a'];
        int a = count['a' - 'a'];
        int l = count['l' - 'a'] / 2;
        int o = count['o' - 'a'] / 2;
        int n = count['n' - 'a'];
        
        int min = b;
        min = Math.min(min, a);
        min = Math.min(min, l);
        min = Math.min(min, o);
        min = Math.min(min, n);
        
        return min;
    }
}
```

---

# Pattern Recognition Cheat Sheet

## Which Template Do I Use?

| Signal in the Problem | Template | Examples from Above |
|---|---|---|
| "Find two elements that sum to X" (unsorted) | Complement Search (HashMap) | 01 |
| Find first unique, find duplicates, count elements | Frequency Map | 02, 03, 04, 05 |
| Input restricted to "lowercase letters" | Array Map (`int[26]`) | 02, 03, 05 |
| Input restricted to "ASCII characters" | Array Map (`int[128]` or `int[256]`) | 04 |

## The Universal Bridge Formula (Hashing)

**Brute Force → Optimal always follows this reasoning chain:**

1. **Brute force** = checking elements against the rest of the array (nested loops) → O(n²)
2. **Ask: "What am I repeatedly searching for?"**
   - Am I searching for a specific complement? → HashMap
   - Am I repeatedly counting characters? → Frequency Map
3. **Ask: "What are my constraints?"**
   - Are the characters limited to the alphabet? → Throw away the `HashMap` and use a primitive `int[]` array for massive speed gains.
4. **Hashing works because** it trades a little bit of Space ($O(N)$ or $O(1)$) to turn an $O(N)$ inner loop search into an instant $O(1)$ lookup.
