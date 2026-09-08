# Fast & Slow Pointer — Revision Sheet

---

## 01. Linked List Cycle

**In My Words:** Given the head of a linked list, determine if it has a cycle (an infinite loop). Return true or false.

**Constraint Whispers:** 
- N ≤ 10⁴ → O(N) time passes easily.
- Classic follow-up demands O(1) space, so external memory structures are out.

**Brute Force:** Store every visited node in a HashSet. If you see a node again, there's a cycle. O(N) time, O(N) space.

**Why It Hurts:** O(N) space violates the standard O(1) space constraint for Linked Lists.

**The Bridge:** If two runners are on a track, and one is twice as fast as the other, the fast runner will eventually "lap" the slow runner IF the track is a circle. If it's a straight line, the fast runner just hits the end. We don't need to remember the past, we just need two pointers moving at different speeds.

**Optimized Intuition:** Slow pointer takes 1 step, fast pointer takes 2 steps. If `fast == slow` at any point, a cycle exists (return true). If `fast` or `fast.next` hits null, the list ends (return false).

**Template:** Fast & Slow Pointers (Floyd's Cycle-Finding)

**Gotcha:** (1) You MUST check both `fast != null && fast.next != null` in the while loop condition to avoid NullPointerException. (2) Check if `slow == fast` (comparing memory addresses), NOT `slow.val == fast.val` (comparing values).

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null) return false;
        
        ListNode slow = head;
        ListNode fast = head;
        
        // "fast khatam na ho jaye !!!"
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            // If they collide, there is a cycle
            if (slow == fast) {
                return true;
            }
        }
        
        // If the loop finishes, fast hit the end of the list
        return false;
    }
}
```

---

## 02. Linked List Cycle II

**In My Words:** Find the exact node where a cycle begins in a linked list. Return the Node itself, or null if no cycle exists.

**Constraint Whispers:** 
- N ≤ 10⁴ → O(N) time expected.
- Follow up explicitly demands O(1) space.

**Brute Force:** Traverse the list, putting every node in a HashSet. The first node you encounter that is *already* in the set is the cycle's start. O(N) time, O(N) space.

**Why It Hurts:** Fails the O(1) space follow-up requirement.

**The Bridge:** Finding the start of a cycle requires a specific two-step mathematical trick (Floyd's algorithm). After the `fast` and `slow` pointers collide to prove a cycle exists, if you reset one pointer back to the `head` and then move BOTH at the same speed (1 step at a time), they are mathematically guaranteed to collide exactly at the cycle entrance!

**Optimized Intuition:** Step 1: `slow` takes 1 step, `fast` takes 2 steps until collision. Step 2: Teleport a new pointer `start` to `head`. Step 3: Move `start` and `slow` forward 1 step at a time until they meet. The meeting node is the cycle entrance.

**Template:** Fast & Slow Pointers + Cycle Entrance Reset

**Gotcha:** (1) Don't store `node.val` in the HashSet if you do the brute force, store the actual `Node` reference! (2) The algorithm is Floyd's Cycle-Finding Algorithm, not Floyd-Warshall (which is a graph shortest-path algorithm).

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        // Step 1: Handle edge cases
        if (head == null) return null;
        
        ListNode slow = head;
        ListNode fast = head;
        
        // Step 2: Find the collision point
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            // We found a cycle!
            if (slow == fast) {
                // Step 3: Reset one pointer to head (let's use a new 'start' pointer)
                ListNode start = head;
                
                // Step 4: Move both at the same speed until they collide again
                while (start != slow) {
                    start = start.next;
                    slow = slow.next;
                }
                
                // The exact node where they collide is the cycle's start point
                return start;
            }
        }
        
        // Fast reached the end of the list, no cycle.
        return null;
    }
}
```

---

## 03. Happy Number

**In My Words:** Replace a number with the sum of the squares of its digits. Repeat until it becomes 1 (happy) or gets trapped in an infinite cycle (not happy).

**Constraint Whispers:** 
- N ≤ 2³¹ - 1 → Standard 32-bit `int` is perfectly fine; it will not overflow. No need for `long`.
- Ideal space complexity should be O(1).

**Brute Force:** Use a HashSet to record each generated sum. If a sum is already in the set, we are in a loop (return false). If we hit 1, return true. O(log N) space.

**Why It Hurts:** The HashSet approach wastes space.

**The Bridge:** We can treat the math sequence like a Linked List! The "next" pointer is simply the math function that calculates the sum of the squares of the digits. Since we just jump from number to number and want to detect a cycle, this is the exact same problem as Linked List Cycle.

**Optimized Intuition:** Use a `getNext(n)` helper function. `slow` calculates the next number once. `fast` calculates it twice (`getNext(getNext(fast))`). If `fast` hits 1, it's happy. If `slow == fast`, they collided in a cycle (not happy).

**Template:** Fast & Slow Pointers (Math sequence as a Linked List)

**Gotcha:** (1) Math trick to split digits: `d = n % 10` gets the last digit, `n = n / 10` chops it off. (2) You MUST give `fast` a head start (`int fast = getNext(n)`) before the while loop, or the condition `slow != fast` instantly fails on the first check.

**Time:** O(log N) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public boolean isHappy(int n) {
        int slow = n;
        // Fast starts one step ahead to prevent an immediate collision on the first check
        int fast = getNext(n);
        
        // Keep running until fast reaches 1 (happy) or they collide (sad loop)
        while (fast != 1 && slow != fast) {
            slow = getNext(slow);
            fast = getNext(getNext(fast));
        }
        
        return fast == 1;
    }
    
    // Helper function to extract digits and sum their squares
    private int getNext(int n) {
        int totalSum = 0;
        
        while (n > 0) {
            int d = n % 10;          // Extract the last digit
            n = n / 10;              // Chop off the last digit
            totalSum += d * d;       // Square it and add to total
        }
        
        return totalSum;
    }
}
```

---

## 04. Find the Duplicate Number

**In My Words:** Given an array of n + 1 numbers (each between 1 and n), find the one repeated number. You CANNOT modify the array and MUST use O(1) space.

**Constraint Whispers:** 
- Pigeonhole Principle: N boxes (range 1 to n) and N+1 pigeons (the array elements) means at least one duplicate is mathematically guaranteed.
- "Without modifying" → NO sorting allowed!
- "O(1) extra space" → NO HashSet allowed!

**Brute Force:** Use a HashSet to track seen numbers. If a number is already in the set, it's the duplicate. O(N) space.

**Why It Hurts:** O(1) space is explicitly mandated.

**The Bridge:** The array values are guaranteed to be valid indices (values are 1 to n, length is n+1). Because of this, we can treat the array exactly like a Linked List, jumping via `current = nums[current]`. Since multiple indices hold the duplicate value, they will all "point" to the same destination index. This creates a cycle. The duplicate number IS the start of the cycle. This perfectly reduces to Linked List Cycle II!

**Optimized Intuition:** Treat the array as a graph. Step 1: Find cycle collision using `slow = nums[slow]` and `fast = nums[nums[fast]]`. Step 2: Reset `slow` to index 0 (which is guaranteed to be outside the cycle). Step 3: Move both pointers at 1x speed (`slow = nums[slow]`). Where they meet is the duplicate number!

**Template:** Fast & Slow Pointers + Cycle Entrance Reset (Array as Linked List)

**Gotcha:** (1) You must use a `do { ... } while (slow != fast);` loop so they take one jump before the collision check. (2) Array jump syntax is `slow = nums[slow]`, not `slow.next`.

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow = 0;
        int fast = 0;
        
        // Step 1: Find the intersection point of the two runners.
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);
        
        // Step 2: Find the "entrance" to the cycle.
        slow = 0;
        
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        
        return slow;
    }
}
```

---

## 05. Middle of the Linked List

**In My Words:** Given a linked list, return the middle node. If it's even length, return the *second* middle node.

**Constraint Whispers:** 
- N ≤ 100 → Extremely small, O(N) is virtually instantaneous.
- Expected to use O(1) space.

**Brute Force:** Pass 1: Walk the list to count the total nodes `N`. Calculate middle index `N / 2`. Pass 2: Walk the list again stopping at `N / 2`. O(N) time, O(1) space.

**Why It Hurts:** It requires 1.5 passes. We can do it in exactly 1 pass.

**The Bridge:** If two runners race and Runner B is TWICE as fast as Runner A, when Runner B hits the finish line, Runner A will be standing exactly at the halfway mark! 

**Optimized Intuition:** `slow` takes 1 step, `fast` takes 2 steps. By the time `fast` reaches the end of the list (`fast == null` or `fast.next == null`), `slow` will be pointing exactly at the middle node. 

**Template:** Fast & Slow Pointers (Tortoise and Hare)

**Gotcha:** (1) Always check `fast != null && fast.next != null` to avoid NullPointerException on the 2-step jump. (2) Starting both at `head` naturally handles the "second middle node for even lengths" edge case perfectly — no extra math needed!

**Time:** O(N) | **Space:** O(1)

**Code Solution:**
```java
class Solution {
    public ListNode middleNode(ListNode head) {
        // Both start at the beginning of the list
        ListNode slow = head;
        ListNode fast = head;
        
        // Loop runs as long as fast hasn't reached the absolute end (null)
        // AND fast isn't standing on the very last node (fast.next != null)
        while (fast != null && fast.next != null) {
            slow = slow.next;        // Tortoise moves 1 step
            fast = fast.next.next;   // Hare moves 2 steps
        }
        
        // When the hare finishes, the tortoise is perfectly in the middle
        return slow;
    }
}
```

---

# Pattern Recognition Cheat Sheet: Fast & Slow Pointers

## The Universal Bridge Formula

**Brute Force → Optimal always follows this reasoning chain:**

1. **Brute force** = Tracking visited elements with a HashSet (O(N) space) or doing multiple passes.
2. **Ask: "Can I do this without remembering the past?"**
   - Is it a loop/cycle? → A fast runner will always lap a slow runner.
   - Do I need the middle? → A 1x runner reaches the middle when a 2x runner reaches the end.
   - Is the array secretly a graph? → If values are valid indices, `nums[current]` acts like `current.next`.

## The Three Core Variants

| Variant | Purpose | Key Logic | Examples |
|---|---|---|---|
| **Cycle Detection** | Find if an infinite loop exists | `slow = slow.next`, `fast = fast.next.next`. If `slow == fast`, cycle exists. | 01, 03 |
| **Cycle Entrance (Floyd's)** | Find the exact node where cycle starts | After collision, reset `slow = head`. Move both at 1x speed. Where they meet is the start. | 02, 04 |
| **Middle/Tortoise & Hare** | Find halfway point in one pass | When `fast` hits the end, `slow` is in the middle. | 05 |

## The Golden Rules

1. **Null Check:** ALWAYS guard the fast pointer's double jump: `while (fast != null && fast.next != null)`.
2. **The `do-while` trick:** If `slow` and `fast` start at the same value in a cycle-entrance problem (like arrays), use a `do-while` loop so they jump once before comparing.
3. **Values vs Objects:** In Linked Lists, check memory addresses (`slow == fast`), NOT values (`slow.val == fast.val`).
