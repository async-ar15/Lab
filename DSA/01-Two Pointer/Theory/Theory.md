# Theory - Two Pointers pattern

We use two pointers/references to traverse or compare elements in a controlled way, so we can avoid unnecessary nested loops. Using this approach, you can reduce the time complexity of many array and string problems from $O(n^2)$ to $O(n)$, or to $O(n \log n)$ when sorting is required first.

## What is a Pointer?
A pointer is simply a variable that represents an index or position within a data structure, such as an array or linked list. The pointers can represent:
- Boundaries of a range (left and right ends)
- Current position and a comparison position (for removing duplicates)
- Two separate arrays (for merging or comparing)
- Fast and slow traversal speeds (for cycle detection)

## Where is it commonly used?
A Two-Pointer algorithm is generally applied to linear data structures:
- Arrays
- Strings
- Linked Lists

Especially when the data is sorted or can be sorted.

## Strong signals to look for (When to use Two Pointers)

When you see these, consider Two Pointers first. It fits problems where the input data follows a predictable pattern:
1. **Sorted array with pair/triplet finding:** Find a pair with a given condition (e.g. Two Sum II), Triplet / Quadruplet (e.g. 3Sum).
2. **In-place array modification:** Remove duplicates, rearrange or partition elements without using extra space (e.g. Move Zeroes, Sort Colors).
3. **Palindrome or symmetry checking:** Compare elements from both ends to see if they converge symmetrically.
4. **Merging or comparing sorted sequences:** Merge two sorted arrays/lists, or compare elements from two ends.
5. **Subarray problems with monotonic conditions:** Closest pair / closest sum, Container / maximum area type problems (e.g. Container With Most Water).

## When NOT to use Two Pointers
- The array is not sorted and sorting would change the answer (e.g., you need to return original indices).
- There is no monotonic relationship between pointer positions and the condition.
- You need to find *all* possible pairs, not just check for existence or find the optimal one (which might still inherently require $O(n^2)$).
- The problem requires looking at non-contiguous elements in arbitrary combinations.

## Sorting is a big clue

If the array/string can be sorted, Two Pointers often becomes much easier because sorting gives us a direction for pointer movement.

**When sorting is required:**
Opposite-direction two pointers for pair-sum problems requires the array to be sorted. It relies on *monotonicity*: increasing left only increases the sum, and decreasing right only decreases it. Without that property, you cannot decide which pointer to move after a comparison.
When sorting is required and the input is unsorted, the overall complexity becomes $O(n \log n)$ for the sort plus $O(n)$ for the two-pointer scan.

**But remember:**
Two Pointers does NOT always require sorted data! For example, opposite-direction for palindrome checking relies on position symmetry, not value order. Same-direction variants (like read/write partitions) also generally do not require sorting.

## Common Two-Pointer structures & Variants

### 1. Opposite direction (Converging Pointers)

```text
L →        ← R
[1  2  3  4  5  6]
```
One pointer starts at the beginning, the other at the end, and they move towards each other. They adjust positions based on comparisons until a condition is met or they cross.

**Think:**
- Pair Sum
- Palindrome
- Reverse
- Container With Most Water
- 3Sum / 4Sum

**Template (Java):**
```java
public void oppositeDirectionTemplate(int[] nums) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right) {
        // Calculate or check current state
        int current = calculateSomething(nums, left, right);

        if (/* found answer */) {
            // Process and possibly return
            return;
        } else if (/* need to increase value */) {
            left++;
        } else {
            right--;
        }
    }
}
```
*Time Complexity: $O(n)$. Space Complexity: $O(1)$.*

### 2. Same direction (Parallel Pointers), but NOT Fast & Slow

```text
first → 
second → 
[1  2  3  4  5  6]
```
Both pointers start at the same end and move in the same direction. They serve complementary roles:
- the **left pointer** is the write pointer or boundary, tracking progress and marking where valid elements end.
- the **right pointer** is the read or explore pointer, scanning ahead looking for the next element to process.

**Think:**
- Removing duplicates (e.g. Remove Duplicates from Sorted Array)
- Rearranging elements (e.g. Move Zeroes)
- In-place modifications / Partitioning array

**Template (Java):**
```java
public int sameDirectionTemplate(int[] nums) {
    int left = 0;  // Write pointer or boundary of valid region
    int result = 0;

    for (int right = 0; right < nums.length; right++) {
        // Check if current element should be kept
        if (shouldKeep(nums[right])) {
            int current = calculateSomething(nums, left, right);
            updateResult();
            left++;
        }
    }

    // left now represents the count of valid elements
    return result;
}
```
*Time Complexity: $O(n)$. Space Complexity: $O(1)$.*

### 3. Fixed-Offset Pointers (Same Direction)
Both pointers move in the same direction, but one is advanced first to create a fixed gap between them before they move together. 

**Think:**
- Processing elements in stages
- Maintaining a fixed distance between pointers (e.g., Finding the Nth node from the end of a linked list)

**Template (Java):**
```java
public void triggerBasedTemplate(int[] nums) {
    int first = 0;
    int second = 0;
    boolean triggered = false;

    while (first < nums.length) {
        // 1) TRIGGER PHASE: advance first until a condition becomes true
        if (!triggered) {
            if (meetsCondition(nums[first])) {
                triggered = true;
                second = 0; // or second = someStartIndex
            }
            first++;
            continue;
        }

        // 2) COUPLED PHASE: now advance second (or both) to extract info
        while (second < first) {
            processPair(nums, second, first - 1);
            second++;
        }

        // optional: reset trigger if the stage ends
        triggered = false;
    }
}
```

### 4. One pointer per sorted sequence
```text
A: [1, 3, 5, 7]
    ↑

B: [2, 3, 6, 8]
    ↑
```
**Think:**
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
