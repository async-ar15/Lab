# Theory - Hashing Pattern

The Hashing pattern relies on using Hash Maps or Hash Sets to achieve **$O(1)$ time complexity** for lookups. By trading space (using $O(N)$ extra memory) for time, we can optimize $O(N^2)$ brute-force array/string solutions down to $O(N)$.

## What is the Hashing Pattern?
Instead of iterating through a dataset multiple times to find complements, pairs, or duplicates, we iterate through it *once*, storing elements in a Hash Map/Set as we go. When we need to look back at elements we've already seen, we can do so instantly in $O(1)$ time.

## Where is it commonly used?
- Arrays
- Strings
- When you need to count frequencies of elements.
- When you need to quickly check if you have seen an element before.

## Strong signals to look for (When to use Hashing)

1. **"Find a pair/elements that sum to X" (Unsorted Data):** If the array is unsorted and you need a pair, sorting takes $O(N \log N)$. Hashing takes $O(N)$. (e.g. Two Sum)
2. **"Count the frequencies":** When the problem asks for the most frequent, least frequent, or exact counts of characters/numbers. (e.g. Valid Anagram, Ransom Note)
3. **"Find duplicates" / "Find unique elements":** Any time you need to group items or ensure uniqueness. (e.g. Contains Duplicate, First Unique Character)
4. **"Group by a specific property":** When elements share a signature and need to be grouped together. (e.g. Group Anagrams)

## When NOT to use Hashing
- **Strict $O(1)$ space constraints:** If the problem explicitly says "do this in $O(1)$ extra space", you cannot use a Hash Map/Set. You'll likely need Two Pointers or sorting.
- **Ordered Data:** Hash Maps and Sets do NOT maintain the order of elements. If you need to find the "closest" value or iterate in ascending order, a BST (like `TreeMap`) or sorting is better.
- **When elements are bounded in a very small range:** If the problem says "only lowercase English letters", a simple integer array of size 26 is much faster and uses less memory overhead than a full `HashMap`! (This is a specialized form of hashing).

## Common Hashing Structures & Variants

### 1. The Frequency Map (Counting)
You iterate through the array/string and count how many times each element appears.

**Think:**
- Valid Anagram
- Ransom Note
- Majority Element

**Template (Java):**
```java
public void frequencyMapTemplate(int[] nums) {
    HashMap<Integer, Integer> counts = new HashMap<>();
    
    for (int num : nums) {
        // getOrDefault is your best friend here!
        counts.put(num, counts.getOrDefault(num, 0) + 1);
    }
    
    // Process the frequencies
    for (int key : counts.keySet()) {
        int freq = counts.get(key);
        // do something based on frequency
    }
}
```

### 2. The Complement Search (Remembering)
Instead of searching for an element ahead of you, you store elements you've already seen. As you process a new element, you check if its "complement" is already in your map.

**Think:**
- Two Sum
- Longest Consecutive Sequence

**Template (Java):**
```java
public int[] complementTemplate(int[] nums, int target) {
    HashMap<Integer, Integer> seen = new HashMap<>(); // val -> index
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        
        if (seen.containsKey(complement)) {
            // Found the pair!
            return new int[] { seen.get(complement), i };
        }
        
        // Didn't find it, add current to 'seen' for future elements
        seen.put(nums[i], i);
    }
    return new int[]{};
}
```

### 3. The Array Map (Small Bound Constraint)
If the data is restricted (e.g., lowercase letters 'a' to 'z'), don't use a `HashMap`. Use a fixed-size array. It acts exactly like a Hash Map but is significantly faster.

**Template (Java):**
```java
public void arrayMapTemplate(String s) {
    int[] count = new int[26]; // for 'a' to 'z'
    
    for (char c : s.toCharArray()) {
        count[c - 'a']++; // 'a' becomes index 0, 'b' becomes index 1, etc.
    }
    
    for (int i = 0; i < 26; i++) {
        if (count[i] > 0) {
            // process character
        }
    }
}
```

# Your interview cheat code

```text
1. Are we checking for existence or counting frequencies?
                  ↓
2. Is the data unsorted? (If sorted, maybe Two Pointers is better)
                  ↓
3. Can we afford O(N) extra space?
                  ↓
4. Is the data limited to a small character set? (Use an int[26] array instead!)
                  ↓
           HASH MAP / SET!
```
