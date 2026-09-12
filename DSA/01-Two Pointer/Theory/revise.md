# Two Pointer — Revision Sheet

---

## 01. Two Sum II - Input Array Is Sorted

**In My Words:** Given a sorted array and a target number, find two numbers that add up to the target. Return their 1-based indices.

**Constraint Whispers:** 
- n ≤ 30,000 → O(n²) does 900 million ops → TLE guaranteed
- "Constant extra space" → HashMap is explicitly killed, must be O(1) space
- Array is SORTED in non-decreasing order → this is the single biggest clue

**Brute Force:** Nested loops trying every pair. O(n²) time, O(1) space.

**Why It Hurts:** 30,000² = 900,000,000 operations. TLE. Also, while a HashMap would fix the time to O(n), the problem explicitly bans extra space — so HashMap is also dead.

**The Bridge:** The array is SORTED. Sorting gives us MONOTONICITY — if I pick the smallest number (left end) and the largest number (right end) and their sum is too big, I KNOW moving right inward will make the sum smaller. If the sum is too small, moving left inward makes it bigger. I always have a clear DIRECTION to move. This is impossible without sorting.

**Better Approach:** HashMap gives O(n) time but O(n) space — killed by the "constant extra space" constraint.

**Optimized Intuition:** Left pointer at index 0 (smallest), right pointer at last index (largest). Compute sum. Too small → left++. Too big → right--. Equal → found the pair.

**Template:** Opposite Direction (Converging Pointers)

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int[] twoSum(int[] numbers, int target) { 
        int left = 0; 
        int right = numbers.length - 1;

        while (left < right) {
            int sum = numbers[left] + numbers[right];

            if (sum == target) {
                return new int[]{left + 1, right + 1}; 
            }
            else if (sum < target) {
                left++; 
            }
            else {
                right--; 
            }
        }
        return new int[] {-1, 1}; 
    }
}
```

---

## 02. Segregate 0s and 1s

**In My Words:** Given an array of only 0s and 1s, rearrange it in-place so all 0s come before all 1s.

**Constraint Whispers:** 
- n ≤ 10⁵ → O(n²) does 10¹⁰ ops → TLE
- "In-place" → must be O(1) space, no new array allowed
- Only two possible values (0 and 1) → exploitable structure

**Brute Force:** Just sort the array. O(n log n) time. Works but doesn't exploit the binary nature of the data.

**Why It Hurts:** Sorting is overkill — we only have TWO distinct values. We should be able to do this in O(n) in a single pass.

**The Bridge:** Since there are only two values, any 1 on the left side is "wrong" and any 0 on the right side is "wrong." If I find a misplaced 1 on the left and a misplaced 0 on the right, I just swap them. Left pointer hunts for 1s (wrong elements), right pointer hunts for 0s (wrong elements).

**Better Approach:** Count the 0s in one pass, then overwrite the array — first chunk with 0s, rest with 1s. O(n) time, O(1) space, but requires TWO passes.

**Optimized Intuition:** Left pointer starts at index 0, right pointer at last index. Left skips past correct 0s. Right skips past correct 1s. When both stop, left is pointing at a 1 and right is pointing at a 0 — swap them. Repeat until they cross. ONE pass.

**Template:** Opposite Direction (Converging Pointers) — partition variant

**Gotcha:** The inner while loops that skip 0s/1s MUST also check `left < right` to prevent ArrayIndexOutOfBounds on all-0s or all-1s arrays.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public void segregate0and1(int[] arr) {
        int left = 0;
        int right = arr.length - 1;

        while (left < right) {
// Keep incrementing left index while we see 0 at left
            while (left < right && arr[left] == 0) {
                left++;
            }
// Keep decrementing right index while we see 1 at right
            while (left < right && arr[right] == 1) {
                right--;
            }

// If left < right, we found a 1 on the left and a 0 on the right, so swap them
            if (left < right) {
// Since we know the values are specifically 0 and 1, we can just hardcode the swap
                arr[left] = 0;
                arr[right] = 1;
                left++;
                right--;
            }
        }
    }
}
```

---

## 03. Remove Duplicates from Sorted List

**In My Words:** Given a sorted linked list, delete all duplicate nodes so each value appears only once. Return the modified list.

**Constraint Whispers:** 
- n ≤ 300 → extremely small, even O(n³) passes, but O(n) is natural for list traversal
- List is SORTED → duplicates are always adjacent, no need to search for them
- Node count can be 0 → must handle empty list (null check!)

**Brute Force:** Use a HashSet to track seen values. Traverse the list, skip any node whose value is already in the set. O(n) time but O(n) space.

**Why It Hurts:** The HashSet wastes space. Since the list is sorted, duplicates are sitting right next to each other — we don't NEED a set to find them.

**The Bridge:** SORTED means duplicates are ADJACENT. I can just stand on a node and peek at the next node. If they match → snip the next node out by rewiring pointers. If they don't match → move forward. No extra data structure needed.

**Optimized Intuition:** Use a single pointer `current` starting at head. Compare `current.val` with `current.next.val`. Same value → bypass by setting `current.next = current.next.next` (but DON'T move current — the new next might also be a duplicate!). Different value → safely move `current = current.next`.

**Template:** Single Pointer Traversal (simplified same-direction — only one pointer needed because we compare current with current.next)

**Gotcha:** Two traps: (1) Must check `head == null` upfront to avoid NullPointerException. (2) Loop condition must be `current != null && current.next != null` — checking just `current != null` will crash when you try to access `current.next.val` on the last node.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        // Handle the edge case of an empty list
        if (head == null) {
            return head;
        }

        ListNode current = head;

        // Traverse until we hit the end of the list
        while (current != null && current.next != null) {
            
            // If we found a duplicate...
            if (current.val == current.next.val) {
                // Bypass the duplicate node
                current.next = current.next.next;
            } else {
                // Otherwise, move to the next distinct node
                current = current.next;
            }
        }
        
        return head;
    }
}
```

---

## 04. Remove Duplicates from Sorted Array

**In My Words:** Given a sorted array, remove duplicates in-place so each number appears only once. Shove unique elements to the front. Return the count of unique elements.

**Constraint Whispers:** 
- n ≤ 30,000 → O(n²) is risky (~900M ops), need O(n)
- "In-place" → O(1) space, no HashSet/new array
- Array is SORTED → duplicates are adjacent

**Brute Force:** Use a HashSet to collect unique values, then copy them back into the array. O(n) time but O(n) space — violates the in-place constraint.

**Why It Hurts:** O(n) space is banned. We need O(1) space.

**The Bridge:** SORTED = duplicates are ADJACENT. I need one pointer to mark "where to write the next unique value" (slow/write pointer) and another to scan ahead looking for new unique values (fast/read pointer). When the fast pointer finds something different from what the slow pointer last wrote, copy it over.

**Optimized Intuition:** `i` (slow) starts at 0 — points to the last unique element placed. `j` (fast) starts at 1 — scans forward. If `nums[i] == nums[j]`, j just moves on (boring duplicate). If `nums[i] != nums[j]`, we found a new unique value! Increment i, copy nums[j] to nums[i]. At the end, return `i + 1` (count = index + 1).

**Template:** Same Direction (Read/Write pointers)

**Gotcha:** Don't forget to actually COPY the value (`nums[i] = nums[j]`), not just move the pointer. And return `i + 1`, not `i` — because i is a 0-based index but we need a count.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // Edge case: constraint says length >= 1, but always good practice
        if (nums.length == 0) return 0;

        int i = 0; // Tracks the index of the last unique element placed

        // j is our scout pointer scanning the array
        for (int j = 1; j < nums.length; j++) {
            // If we find a brand new unique element...
            if (nums[i] != nums[j]) {
                i++; // Move i forward
                nums[i] = nums[j]; // Place the new unique element at i
            }
            // If they are equal, j just naturally increments via the for-loop
        }

        // The number of unique elements is the index i + 1
        return i + 1;
    }
}
```

---

## 05. Squares of a Sorted Array

**In My Words:** Given a sorted array (may contain negatives), return a new array of the squares of each number, also sorted.

**Constraint Whispers:** 
- n ≤ 10,000 → O(n log n) passes easily, but the follow-up explicitly dares us to find O(n)
- Array has NEGATIVES → squaring a large negative gives a large positive, which breaks the sorted order
- Must return a NEW array → O(n) space is unavoidable for the output

**Brute Force:** Square every element, then sort the result. O(n log n) time. Trivially correct but doesn't exploit the structure.

**Why It Hurts:** The follow-up says "can you do O(n)?" — the interviewer will not be satisfied with square-then-sort.

**The Bridge:** In a sorted array with negatives, the LARGEST squares are always at the EXTREMES — either the far left (big negative → big square) or the far right (big positive → big square). The smallest squares are somewhere in the middle near zero. So if I compare squares from both ends, I can always pick the larger one.

**Optimized Intuition:** Left pointer at index 0, right pointer at last index. Compare `left² vs right²`. Whichever is BIGGER gets placed at the END of the result array (fill result from right-to-left, index = n-1 down to 0). Move that pointer inward. Repeat until pointers cross.

**Template:** Opposite Direction (Converging Pointers) — filling result array backwards

**Gotcha:** Loop condition must be `left <= right` (not `left < right`). Using strict `<` skips the last element when both pointers meet at the same index — that element never gets squared and placed.

**Time:** O(n) | **Space:** O(n) — for the result array

**Code Solution:**
```java
class Solution {
    public int[] sortedSquares(int[] nums) {
        int n = nums.length;
        int[] result = new int[n]; // O(n) space for the answer
        
        int left = 0;
        int right = n - 1;
        int index = n - 1; // Start at the end of the new array
        
        while (left <= right) {
            int leftSquare = nums[left] * nums[left];
            int rightSquare = nums[right] * nums[right];
            
            // Compare the squares and pick the largest one
            if (leftSquare > rightSquare) {
                result[index] = leftSquare;
                left++; // Move the left pointer inward
            } else {
                result[index] = rightSquare;
                right--; // Move the right pointer inward
            }
            index--; // Move to the next empty spot in our result array
        }
        
        return result;
    }
}
```

---

## 06. 3Sum

**In My Words:** Given an unsorted array, find ALL unique triplets that sum to zero. No duplicate triplets in the output.

**Constraint Whispers:** 
- n ≤ 3,000 → O(n²) = 9M ops → passes easily. O(n³) = 27B ops → TLE
- No space restriction mentioned, but O(1) auxiliary is achievable
- "No duplicate triplets" → the hardest part of the problem

**Brute Force:** Three nested loops checking every combination. Use a HashSet of sorted triplets to filter duplicates. O(n³) time → TLE.

**Why It Hurts:** 3000³ = 27 billion operations. Way too slow.

**The Bridge:** 3Sum is literally Two Sum II in disguise! If I sort the array and fix one number A using a for-loop, the remaining problem becomes: "find two numbers B + C = -A in a sorted array." That's exactly Two Sum II! The sort also makes duplicate-skipping trivial — identical numbers sit next to each other, so I just skip if `nums[i] == nums[i-1]`.

**Optimized Intuition:** Sort the array. For each index `i` (the "A" number): set left = i+1, right = end. Run the standard Two Sum II converging pointers for target = -nums[i]. When a triplet is found, add it, then skip duplicate B values by advancing left past repeated values. Also skip duplicate A values at the for-loop level.

**Template:** Opposite Direction (Converging Pointers) — nested inside a for-loop

**Gotcha:** THREE things to remember: (1) Skip duplicate A's: `if (i > 0 && nums[i] == nums[i-1]) continue`. (2) Skip duplicate B's after a successful hit: `while (left < right && nums[left] == nums[left-1]) left++`. (3) Early termination: if `nums[i] > 0`, break — a sorted array can't sum three positives to zero. (4) After finding a triplet, you MUST move at least one pointer (left++) before the duplicate-skip loop, or you get an infinite loop.

**Time:** O(n²) | **Space:** O(1) auxiliary (ignoring output)

**Code Solution:**
```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        
        // 1. Sort the array so we can use Two Pointers and easily skip duplicates
        Arrays.sort(nums);
        
        for (int i = 0; i < nums.length; i++) {
            // If the first number is > 0, the sum can NEVER be 0 (since array is sorted)
            if (nums[i] > 0) break;
            
            // Skip duplicate 'A' numbers so we don't build duplicate triplets
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            
            // Standard Two Pointer approach for the rest of the array
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                
                if (sum > 0) {
                    right--; // Sum is too big, shrink from the right
                } else if (sum < 0) {
                    left++; // Sum is too small, grow from the left
                } else {
                    // We found a triplet!
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    // We need to keep searching for more pairs for this 'A'
                    left++;
                    
                    // Skip duplicate 'B' numbers so we don't build duplicate triplets
                    while (left < right && nums[left] == nums[left - 1]) {
                        left++;
                    }
                }
            }
        }
        
        return result;
    }
}
```

---

## 07. 3Sum Closest

**In My Words:** Given an array and a target, find three numbers whose sum is closest to the target. Return that sum (not the difference, not the numbers — the sum itself).

**Constraint Whispers:** 
- n ≤ 500 → O(n²) = 250K ops → lightning fast. O(n³) = 125M ops → risky but might barely pass
- "Exactly one solution" → no need to handle ties
- We need to track "closeness" → Math.abs(sum - target)

**Brute Force:** Three nested loops. For every triplet, compute sum. Track the one with the smallest |sum - target|. O(n³) time.

**Why It Hurts:** 125M ops is borderline. The interviewer wants O(n²).

**The Bridge:** Exact same structure as 3Sum — sort + fix one number + two pointers. The only difference: instead of checking `sum == 0`, we track the sum with the smallest absolute distance to target. Pointer movement logic is identical: too small → left++, too big → right--, exact match → return immediately (distance = 0, can't beat that).

**Optimized Intuition:** Sort. For each i: left = i+1, right = end. Compute currentSum. If `|currentSum - target| < |closestSum - target|`, update closestSum. Move pointers: currentSum > target → right--, currentSum < target → left++, currentSum == target → return immediately.

**Template:** Opposite Direction (Converging Pointers) — nested inside a for-loop (3Sum variant)

**Gotcha:** Do NOT initialize closestSum to Integer.MAX_VALUE! If target is negative, `MAX_VALUE - target` overflows and produces garbage. Initialize with `nums[0] + nums[1] + nums[2]` — the first actual valid triplet sum.

**Time:** O(n²) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        
        // Initialize with the sum of the first three elements
        int closestSum = nums[0] + nums[1] + nums[2];
        
        for (int i = 0; i < nums.length; i++) {
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) {
                int currentSum = nums[i] + nums[left] + nums[right];
                
                // The Mathematics: checking the absolute distance to the target
                if (Math.abs(currentSum - target) < Math.abs(closestSum - target)) {
                    closestSum = currentSum;
                }
                
                // Move pointers based on how currentSum compares to the target
                if (currentSum > target) {
                    right--;
                } else if (currentSum < target) {
                    left++;
                } else {
                    // If currentSum == target, distance is 0. We found the perfect answer.
                    return currentSum;
                }
            }
        }
        
        return closestSum;
    }
}
```

---

## 08. Triplets with Smaller Sum

**In My Words:** Given an array of distinct integers and a target sum, COUNT how many unique triplets have a sum strictly less than the target.

**Constraint Whispers:** 
- n ≤ 1,000 → O(n²) = 1M ops → fast. O(n³) = 1B ops → TLE
- "Count" not "list" → we need a number, not actual triplets
- "Strictly less than" → must use `<`, not `<=`

**Brute Force:** Three nested loops. Check every triplet. If sum < target, increment counter. O(n³) time.

**Why It Hurts:** 1000³ = 1 billion operations. TLE.

**The Bridge:** Same 3Sum skeleton — sort + fix one number + two pointers. But the COUNTING trick is the key insight: if `arr[i] + arr[left] + arr[right] < target`, then because the array is sorted, EVERY element between left and right will ALSO form a valid triplet with arr[left]. So instead of counting one-by-one, we add `(right - left)` to count in one shot. This is what makes it O(n²) instead of O(n³).

**Optimized Intuition:** Sort. For each i: left = i+1, right = end. If currentSum < target → the math trick: `count += (right - left)`, then left++ (try a bigger left). If currentSum >= target → right-- (shrink the sum).

**Template:** Opposite Direction (Converging Pointers) — nested inside a for-loop (3Sum counting variant)

**Gotcha:** (1) Don't do `count++` — you'll miss all the elements between left and right. Must do `count += (right - left)`. (2) The condition is STRICTLY less than (`<`), so when sum equals target, treat it as too big and do right--.

**Time:** O(n²) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    long countTriplets(long arr[], int n, int sum) {
        Arrays.sort(arr);
        long count = 0;
        
        for (int i = 0; i < n; i++) {
            int left = i + 1;
            int right = n - 1;
            
            while (left < right) {
                long currentSum = arr[i] + arr[left] + arr[right];
                
                if (currentSum < sum) {
                    // The Mathematics trick:
                    // If arr[right] works with arr[left], then everything 
                    // before right will also work with arr[left] because it's sorted!
                    count = count + (right - left);
                    left++; // Now let's try a bigger left
                } else {
                    // Too big or equal, need to shrink the sum
                    right--;
                }
            }
        }
        
        return count;
    }
}
```

---

## 09. Subarray Product Less Than K

**In My Words:** Given an array of positive integers and k, count the number of CONTIGUOUS subarrays whose product is strictly less than k.

**Constraint Whispers:** 
- n ≤ 30,000 → O(n²) = 900M ops → TLE
- All nums[i] ≥ 1 → products only grow or stay same as window expands (monotonic!)
- "Contiguous subarrays" → CANNOT sort (sorting destroys contiguity)
- k can be 0 → edge case since all products ≥ 1

**Brute Force:** Two nested loops. Outer picks start, inner expands end while maintaining running product. Break when product ≥ k. O(n²) time.

**Why It Hurts:** 900M operations. TLE.

**The Bridge:** "Contiguous subarray" + "product grows monotonically as window expands" = SLIDING WINDOW. When the product gets too big, shrink from the left (divide out the leftmost element). The counting trick: for each position of `right`, the number of NEW valid subarrays ending at `right` is exactly `(right - left + 1)` — that's the current window size.

**Optimized Intuition:** left = 0, runningProduct = 1. For each right: multiply nums[right] into runningProduct. While runningProduct >= k: divide out nums[left] and left++. Then count += (right - left + 1). The window always contains a valid product, and we count all subarrays ending at right.

**Template:** Sliding Window (Same Direction pointers — a close relative of two pointers)

**Gotcha:** (1) If k ≤ 1, return 0 immediately — since all numbers are ≥ 1, no product can be strictly less than 1 (or 0). (2) NEVER sort this array — "contiguous subarrays" means order matters. Sorting is one of the most common wrong instincts here.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        // Edge case: Since all numbers are >= 1, we can never have a product < 1.
        if (k <= 1) return 0;
        
        int count = 0;
        int runningProduct = 1;
        int left = 0;
        
        for (int right = 0; right < nums.length; right++) {
            // Expand the window
            runningProduct = runningProduct * nums[right];
            
            // Shrink the window if the product gets too big
            while (runningProduct >= k) {
                runningProduct = runningProduct / nums[left];
                left++;
            }
            
            // Math trick: The number of valid contiguous subarrays ending at 'right' 
            // is exactly equal to the size of the window!
            count = count + (right - left + 1);
        }
        
        return count;
    }
}
```

---

## 10. Sort Colors (Dutch National Flag)

**In My Words:** Given an array of only 0s, 1s, and 2s, sort it in-place so all 0s come first, then 1s, then 2s. Cannot use a library sort. Follow-up: do it in one pass.

**Constraint Whispers:** 
- n ≤ 300 → tiny, even bubble sort works, but follow-up demands one-pass O(n)
- Only THREE possible values → massively exploitable
- "In-place" + "one-pass" + "constant space" → need a clever partitioning scheme

**Brute Force:** Counting sort — count how many 0s, 1s, 2s in one pass, then overwrite the array in a second pass. O(n) time, O(1) space. Works, but requires TWO passes.

**Why It Hurts:** The follow-up asks: "can you do it in ONE pass?" Counting sort needs two.

**The Bridge:** With only three categories, we need THREE pointers, not two. Think of it as maintaining three zones: [0s region | 1s region | unexplored | 2s region]. `low` marks the boundary of the 0s zone. `high` marks the boundary of the 2s zone. `mid` is the explorer that walks through the unexplored region and tosses elements to the correct zone.

**Optimized Intuition:** Three pointers: low = 0, mid = 0, high = end. While mid ≤ high: if nums[mid] == 0 → swap with low, advance both low++ and mid++. If nums[mid] == 1 → it's already in the right zone, just mid++. If nums[mid] == 2 → swap with high, high-- but DO NOT advance mid (the swapped-in element is unknown and needs to be checked).

**Template:** Three Pointers (Dutch National Flag) — a specialized partition variant

**Gotcha:** THE classic trap: when you swap nums[mid] with nums[high], you must NOT increment mid. The element that just arrived from the high end could be a 0, 1, or 2 — you haven't inspected it yet. If you blindly mid++, you'll skip it and the array won't be fully sorted.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public void sortColors(int[] nums) {
        int low = 0;
        int mid = 0;
        int high = nums.length - 1;
        
        while (mid <= high) {
            if (nums[mid] == 0) {
                // Swap mid and low
                int temp = nums[low];
                nums[low] = nums[mid];
                nums[mid] = temp;
                
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                // Already in the right place
                mid++;
            } else if (nums[mid] == 2) {
                // Swap mid and high
                int temp = nums[high];
                nums[high] = nums[mid];
                nums[mid] = temp;
                
                high--;
                // We DO NOT increment mid here because the new number 
                // swapped from the high position needs to be checked!
            }
        }
    }
}
```

---

## 11. Remove Duplicates from Sorted Array II

**In My Words:** Given a sorted array, remove duplicates in-place so each number appears at most TWICE. Shove valid elements to the front. Return the count.

**Constraint Whispers:** 
- n ≤ 30,000 → need O(n)
- "In-place" + "O(1) extra memory" → no HashMap, no frequency array
- Array is SORTED → duplicates are adjacent, and we can exploit position-based comparisons

**Brute Force:** Use nested loops to count occurrences. When count hits 3, shift all remaining elements left by one to overwrite. O(n²) time because of the shifting.

**Why It Hurts:** Shifting inside a loop is O(n) per shift, done up to O(n) times = O(n²). Too slow.

**The Bridge:** This is Remove Duplicates I (Problem 04) with a twist. In Problem 04, we compared nums[i] with nums[j] to check "is this a new value?". Here, we allow TWO copies. The key insight: compare the current element `nums[i]` not with the previous element, but with the element TWO positions back in the CLEAN array — `nums[k-2]`. If they're the same, we already have two copies. If they're different, it's safe to keep.

**Optimized Intuition:** Start k = 2 (first two elements are always valid — you can't have 3+ duplicates in just 2 slots). For each i from 2 onwards: compare nums[i] with nums[k-2] (two positions back in the CLEAN section). If different → place it at nums[k], k++. If same → skip it (would be a 3rd copy).

**Template:** Same Direction (Read/Write pointers) — generalized duplicate removal

**Gotcha:** Compare against `nums[k-2]`, NOT `nums[i-2]`! `k` represents the clean, rebuilt array. `i` is scanning through the dirty original. You must check against the clean version because the dirty array may still have old values that confuse the comparison.

**Time:** O(n) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        // Edge case: if length is 2 or less, all elements are valid
        if (nums.length <= 2) {
            return nums.length;
        }
        
        // k is the pointer for where to place the next valid number.
        // We start at 2 because index 0 and 1 are always valid (we can have at most 2 duplicates)
        int k = 2; 
        
        for (int i = 2; i < nums.length; i++) {
            // Check against the newly built section of the array (k - 2)
            // If they match, it's a 3rd duplicate. If they don't, it's safe!
            if (nums[i] != nums[k - 2]) {
                nums[k] = nums[i];
                k++;
            }
        }
        
        return k;
    }
}
```

---

# Pattern Recognition Cheat Sheet

## Which Template Do I Use?

| Signal in the Problem | Template | Examples from Above |
|---|---|---|
| Sorted array + find a pair/triplet summing to X | Opposite Direction | 01, 06, 07, 08 |
| In-place remove/deduplicate from sorted data | Same Direction (Read/Write) | 03, 04, 11 |
| Partition array into 2 groups | Opposite Direction (Partition) | 02 |
| Partition array into 3 groups | Three Pointers (Dutch Flag) | 10 |
| Contiguous subarray with monotonic condition | Sliding Window | 09 |
| Largest values at extremes of sorted array | Opposite Direction (fill backwards) | 05 |

## The Universal Bridge Formula

**Brute Force → Optimal always follows this reasoning chain:**

1. **Brute force** = try every combination (nested loops) → O(n²) or O(n³)
2. **Ask: "What structure am I ignoring?"**
   - Is the data SORTED? → Monotonicity lets me decide which pointer to move
   - Are there only K distinct values? → Partition instead of sort
   - Is the condition MONOTONIC? (grows as window expands) → Sliding window
3. **Two pointers work because** they let each element be processed at most once, collapsing an O(n) inner loop into O(1) decisions per step

## The Three Questions Before You Code

1. **Can I sort?** (Will sorting break the answer? e.g., subarrays = NO, pairs = YES)
2. **Where do the pointers start?** (Same end = read/write, opposite ends = converging)
3. **What tells me which pointer to move?** (This is the MONOTONICITY argument — the heart of every two-pointer proof)
