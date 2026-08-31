# Theory - Two Pointers pattern

we use two pointers/references to traverse or compare elements in a controlled way, so we can avoid unnecessary nested loops

## Where is it commonly used?
- Arrays
- Strings
- Linked Lists

Especially when the data is sorted or can be sorted.

## Strong signals to look for

When you see these, consider Two Pointers first:
- Find a pair with a given condition
- Triplet / Quadruplet
- Remove duplicates
- Rearrange elements
- Partition elements
- Reverse
- Palindrome
- Merge two sorted arrays/lists
- Compare elements from two ends
- Closest pair / closest sum
- Container / maximum area type problems

## Sorting is a big clue

If the array/string can be sorted, Two Pointers often becomes much easier because sorting gives us a direction for pointer movement.

**But remember:**
Two Pointers does NOT always require sorted data.

## Common Two-Pointer structures

### 1. Opposite direction

```text
L →        ← R
[1  2  3  4  5  6]
```

Think:
- Pair Sum
- Palindrome
- Reverse
- Container With Most Water
- 3Sum / 4Sum

### 2. Same direction, but NOT Fast & Slow

You can have two pointers moving forward for things like:
- Removing duplicates
- Rearranging elements
- In-place modifications

But don't confuse this with the Fast & Slow Pointer pattern you're studying separately.

### 3. One pointer per sorted sequence
```text
A: [1, 3, 5, 7]
    ↑

B: [2, 3, 6, 8]
    ↑
```

Think:
- Merge
- Intersection
- Comparing two sorted sequences

# Your interview cheat code

```text
1. Is it an Array / String / Linked List?
                  ↓
2. Is there a Pair / Triplet / Quadruplet?
                  ↓
3. Is the data sorted or can sorting help?
                  ↓
4. Am I comparing elements from opposite ends?
                  ↓
5. Am I removing / rearranging / partitioning elements?
                  ↓
6. Am I merging or comparing two sorted sequences?
                  ↓
          TWO POINTERS?
```
