# 387. First Unique Character in a String

## Problem Statement
Given a string `s`, find the **first non-repeating character** in it and return its index. If it does not exist, return `-1`.

## Examples
**Example 1:**
- Input: `s = "leetcode"`
- Output: `0`
- Explanation: The character 'l' at index 0 is the first character that does not occur at any other index.

**Example 2:**
- Input: `s = "loveleetcode"`
- Output: `2`

**Example 3:**
- Input: `s = "aabb"`
- Output: `-1`

## Constraints
- `1 <= s.length <= 10^5`
- `s` consists of only lowercase English letters.

---

# understanding the question 

Pointing out these things:
- i. Draw examples
- ii. Clarify edge cases
- iii. Confirm input/output
- iv. Important key words for the approach 
- v. basic level of understanding of the question & what kinda solution might work for us 

### my understanding
- give S as the string 
- we have to find the 1st non repeating character in it & return its index 
- if it does not exist return -1 
- *Note: I did not get the question espeically after the 3rd example*
---
okay again my understanding is that we have to get the unique elements lets say we have the 
a,b,c,a 
now we see a appear 2x that means not unique 
and b appears 1x that means unique elements that means we have to return the indices of b , also in the same way if all the elements in the array are not unique there same value appear 2x or more than we have to give -1  
---
**Agent Feedback:**
Boom, you nailed it! 🎯 Your example of `a, b, c, a` is perfect.
- `a` appears twice (not unique)
- `b` appears once (unique)
So we return the index of `b` (which is 1) because it's the *first* unique one we hit. And exactly like you said, if they all repeat 2x or more, we return `-1`. You fully understand the problem now!

# understanding the constraints

Pointing out these things: 
- i. Time complexity
- ii. Space complexity
- iii. Input space, output space
- iv. What kind of data structure or algorithm can be used here
- v. how constraints help us to find the solution 

### my understanding
- *not able to extract any thing from the constratints*

**Agent Feedback:**
No worries, let's break them down:
1. `s.length <= 10^5`: This is the big one! If you use a brute force approach (checking every character against every other character), that's $O(N^2)$. $(10^5)^2 = 10^{10}$ operations, which will cause a Time Limit Exceeded (TLE) error. This tells us we **must** find an $O(N)$ solution. 
2. `s consists of only lowercase English letters`: This is a massive hint! There are exactly 26 lowercase English letters. Instead of using a heavy, dynamic `HashMap`, you can use a simple array of size 26 to keep track of character frequencies. (Remember the `Array Map` pattern from our Theory notes?)

# Solution 

## Brute force 

- **Intuition for the brute force**: For every character in the string, scan the entire string again to see if that character appears anywhere else. If we finish the inner loop and didn't find any duplicates for the current character, we return its index!
- **pseudo code for the brute force**:
  ```java
  for i from 0 to s.length() - 1:
      boolean isUnique = true
      for j from 0 to s.length() - 1:
          if i != j and s[i] == s[j]:
              isUnique = false
              break
      if isUnique == true:
          return i
  return -1
  ```
- **draw the dry run for the brute force**: 
  `s = "aabb"`
  - `i = 0` ('a'): Check against `j=1` ('a'). It matches! Not unique. Break.
  - `i = 1` ('a'): Check against `j=0` ('a'). It matches! Not unique. Break.
  - `i = 2` ('b'): Check against `j=3` ('b'). It matches! Not unique. Break.
  - `i = 3` ('b'): Check against `j=2` ('b'). It matches! Not unique. Break.
  - Loop finishes, return `-1`.
- **Time complexity**: $O(N^2)$ because for every character, we potentially scan the rest of the string.
- **Space complexity**: $O(1)$ since we are just using loop variables.
- **solution code**:
```java
// User will write this!
```

---
*User's previous notes:*
hey I did not came up with the brute forece & the better approach I can think directly of the 
optimized that we have to find the unique that means thins click in my head is the hashmap 
key = letter 
value = frequency 
& then we will pass it once asn the thing which appear once we will return its index !!! how about my approach !!!

**Agent Feedback:**
That is exactly the optimized approach! 🧠 Good job instantly recognizing the frequency counting pattern.

## optimised code (Hashing / Array Map)

- **how we are optimising from the brute force**: Instead of repeatedly checking the whole string to see if a character is a duplicate, we can do a **two-pass** approach. We go through the string once to count frequencies, and a second time to find the first character with a frequency of 1.
- **Intuition**: 
  - **Pass 1:** Count the occurrences of each character using a frequency map (or a size-26 integer array since it's just lowercase letters). 
  - **Pass 2:** Iterate through the string from left to right. Check the count of the current character. The first one we find that has a count of exactly `1` is our answer!
- **pseudo code for the optimised approach**:
  ```java
  int[] count = new int[26]  // size 26 for 'a' to 'z'
  
  // Pass 1: Build frequency map
  for (int i = 0; i < s.length(); i++) {
      char c = s.charAt(i);
      count[c - 'a'] = count[c - 'a'] + 1;
  }
      
  // Pass 2: Find the first unique character
  for (int i = 0; i < s.length(); i++) {
      char c = s.charAt(i);
      if (count[c - 'a'] == 1) {
          return i;
      }
  }
          
  return -1;
  ```
- **draw the dry run for the optimised approach**:
  `s = "leetcode"`
  - **Pass 1:**
    - `l`: count['l'-'a'] = 1
    - `e`: count['e'-'a'] = 1
    - `e`: count['e'-'a'] = 2
    - `t`: count['t'-'a'] = 1
    - `c`: count['c'-'a'] = 1
    - `o`: count['o'-'a'] = 1
    - `d`: count['d'-'a'] = 1
    - `e`: count['e'-'a'] = 3
  - **Pass 2:**
    - `i = 0`, char = `l`. `count['l'-'a'] == 1`. It's 1! 
    - We immediately return `0`.
- **Time complexity**: $O(N)$ because we loop through the string exactly twice. Operations inside the loops are $O(1)$.
- **Space complexity**: $O(1)$ because the `count` array is always exactly size 26, regardless of how large the string gets.
- **solution code**:
```java
// You're up! Use the pseudo-code above to write the actual Java code here!
```

# question where I went wrong & what is the correction 

**Agent Feedback:**
Almost there, but we have a few critical bugs in the code! Let's track them for your corrections:
1. **Nested Loops vs Sequential Loops:** Look closely at your `for` loops. You put the second loop *inside* the first loop's brackets. This makes it $O(N^2)$ again! We need two *separate* passes, one after the other.
2. **Populating the Map:** In your first loop (typo `fro`), you never actually put anything into the `map`! You need to use `map.put(ch, map.getOrDefault(ch, 0) + 1)` to actually count the frequencies.
3. **Variable Redeclaration:** You declared `char ch` in the first loop, and then tried to declare `char ch = s.charAt(i)` again inside the second loop. 
4. **Return Value:** The method expects an `int` (the index), but you just wrote `return;` inside the `if` statement. It needs to be `return i;`.

Take another shot at writing the code based on these corrections! Remember, two separate passes!

class Solution {

    public int firstUniqChar(String s)
    {

        HashMap <Character, Integer> map = new HashMap<>();

        fro (char ch : s.toCharArray()){
            for (int i = 0; i < s.length(); i++){
                char ch = s.charAt(i);
                if(map.get(ch) == 1){
                    return;
                }
            }
        }
        return -1;
    }

}