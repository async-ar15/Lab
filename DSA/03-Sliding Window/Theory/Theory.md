# Theory - Sliding Window Pattern

Sliding window is a problem-solving pattern where you use two pointers to define a "window" and slide them over a data structure, typically an array or a string, to find subarrays or substrings that meet a certain requirement.

This approach can reduce the time complexity of many array and string problems from $O(n^2)$ to $O(n)$.

## What is Sliding Window?
A sliding window is a technique that maintains a subset of elements from an array or string, moving through the data one position at a time. As the window slides, we update our result incrementally rather than recomputing from scratch.

The window is defined by two pointers:
- **Left pointer (start):** Marks the beginning of the window
- **Right pointer (end):** Marks the end of the window

Updates happen incrementally. When sliding from one position to the next:
- We lose one element (the one leaving the window on the left)
- We gain one element (the one entering the window on the right)
- Everything else stays the same

The window in a Sliding Window algorithm expands, shrinks, and slides over a data structure:
- **Expanding the window:** Moving the right pointer forward to include more elements
- **Shrinking the window:** Moving the left pointer forward to exclude elements when constraints are violated
- **Sliding the window:** Adjusting both left and right pointers as needed while maintaining a valid state

## Why Sliding Window Works (Amortized $O(n)$)
Sliding window relies on a simple observation: consecutive windows overlap significantly. If we have already computed something for window `[i, i+k-1]`, the next window `[i+1, i+k]` shares `k-1` elements with the previous one. We only need to account for one element leaving and one element entering, instead of recomputing everything.

The variable-size template has a `for` loop with a `while` loop nested inside it. At first glance this looks like $O(n^2)$, but it's actually $O(n)$.
- The outer `for` loop advances `right` from 0 to $n - 1$. That's exactly $n$ iterations.
- The inner `while` loop advances `left`. Across the entire execution of the algorithm, `left` also moves from 0 to at most $n - 1$. That's at most $n$ iterations TOTAL across all outer iterations combined, not $n$ iterations per outer iteration.
Total work: $n$ outer iterations + at most $n$ inner iterations = $O(n)$.

## Two Variants of Sliding Window

### 1. Fixed Size Window
Fixed-size window problems follow a predictable pattern. We first build the initial window of size $k$, then slide it through the array by removing the leftmost element and adding the next element on the right.

**Template (Java):**
```java
public int[] fixedWindowTemplate(int[] nums, int k) {
    int n = nums.length;
    if (n < k) return new int[0];

    // Step 1: Build the initial window of size k
    int windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += nums[i];
    }

    // Process the first window
    int maxSum = windowSum;

    // Step 2: Slide the window from index k to n-1
    for (int i = k; i < n; i++) {
        // Add the new element entering the window
        windowSum += nums[i];

        // Remove the element leaving the window
        windowSum -= nums[i - k];

        // Process the current window
        maxSum = Math.max(maxSum, windowSum);
    }

    return new int[]{maxSum};
}
```

### 2. Variable Size Window
The window grows and shrinks based on a condition. We use two pointers, `left` and `right`, where `right` expands the window and `left` contracts it. The general pattern is: expand until a condition is violated, then shrink until the condition is restored.

**Template (Longest Valid Window - Java):**
```java
public int longestValidWindow(int[] nums, int targetCondition) {
    int n = nums.length;
    int left = 0;
    int result = 0;
    int windowState = 0;

    for (int right = 0; right < n; right++) {
        windowState += nums[right];  // Add nums[right] to window state
        
        // Shrink window while condition is violated
        while (conditionViolated(windowState, targetCondition)) {
            windowState -= nums[left];
            left++;
        }

        // Process valid window
        result = Math.max(result, right - left + 1);
    }

    return result;
}
```

**Template (Shortest Valid Window - Java):**
```java
public int shortestValidWindow(int[] nums, int targetCondition) {
    int left = 0, result = Integer.MAX_VALUE;
    int windowState = 0;
    
    for (int right = 0; right < nums.length; right++) {
        windowState += nums[right];
        
        while (windowIsValid(windowState, targetCondition)) {
            result = Math.min(result, right - left + 1);
            windowState -= nums[left];
            left++;
        }
    }
    return result;
}
```

## The `atMost` Trick for "Exactly K" Problems
Some sliding window problems ask you to count subarrays where some property holds exactly $K$ times (e.g., Subarrays with K Different Integers). A direct sliding window for "exactly K" is awkward because the "exactly K" predicate isn't monotone with respect to window size.

**The Fix:** Define `atMost(K)` as the number of subarrays where the property holds at most $K$ times. This IS solvable with sliding window, since the "at most K" predicate is monotone: shrinking a window can only decrease the count.
Then the answer is: `exactly(K) = atMost(K) - atMost(K - 1)`

## Choosing the Window State
What changes from problem to problem is what we track inside the window:
- Max sum of size-k subarray -> One integer (running sum)
- Longest substring with K distinct characters -> HashMap from character to count
- Find All Anagrams -> `int[26]` frequency array
- Longest substring without repeating characters -> HashSet, or `int[128]` of last-seen indices
- Sliding Window Maximum -> Monotonic deque

## Strong signals to look for (When to use)
Reach for sliding window when you see these patterns:
1. **Contiguous subarray or substring:** Elements must be next to each other.
2. **Some form of "longest," "shortest," "maximum," or "minimum"**
3. **The "expand and shrink" mental model applies:** You expand until a condition breaks, then shrink until it is satisfied again.
4. **$O(n)$ solution seems possible:** If brute force is $O(n^2)$ and you suspect $O(n)$ is achievable.

## When NOT to use Sliding Window
- Elements need not be contiguous (use dynamic programming or other techniques).
- Array contains negative numbers and you need exact sums (use prefix sum + hashmap instead, because adding a negative doesn't monotonically increase the sum).
- You need to track all subarrays, not just optimal ones.
- The order of elements can be rearranged.
