# Fast & Slow Pointer Pattern Theory

The Fast and Slow pointer pattern (often called the **Hare and Tortoise Algorithm** or **Floyd's Cycle-Finding Algorithm**) is a clever technique where you use two pointers that travel through a structure (usually a Linked List) at different speeds.

**Real-World Analogy**
Imagine two runners, A and B, on a circular track. Runner A is running faster than Runner B. Since the track is circular, at some point, Runner A will catch up to and pass Runner B. This is essentially how the Fast & Slow Pointers pattern works. 
- **Slow pointer:** Moves one step at a time
- **Fast pointer:** Moves two steps at a time

By having one pointer move faster than the other, you can solve extremely common types of problems efficiently, using only $O(n)$ time and $O(1)$ space complexity, often in just one pass through the data.

## When to use Fast and Slow Pointers
1. **Finding the middle element:** Middle of the Linked List, Reorder List
2. **Cycle detection:** Linked List Cycle, Linked List Cycle II
3. **Finding the start of a cycle:** Once detected, mathematically finding the entry node.
4. **Checking for palindromes in linked lists:** Combined with list reversal, finding the middle helps check if a linked list is a palindrome.
5. **Cycle in sequences (numbers/arrays):** Some numerical problems involve sequences that might cycle (e.g., Happy Number, Find the Duplicate Number).
6. **List partitioning:** Split list into halves for merge sort.

---

## 1. Finding the Middle of a Linked List

If you don't know how long a Linked List is, how do you find the middle?
You could loop through the whole thing to count the length, then loop through half of it again... but there is a better way!

**The Trick:**
- `slow` moves 1 step at a time.
- `fast` moves 2 steps at a time.
By the time the `fast` pointer reaches the very end of the list, the `slow` pointer will be exactly halfway there!

```text
slow = head
fast = head

WHILE fast is not null AND fast.next is not null:
    slow = slow.next       // 1 step
    fast = fast.next.next  // 2 steps
    
// slow is now exactly at the middle node!
```

**Template (Java):**
```java
public ListNode findMiddle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    // Move until fast reaches the end
    while (fast != null && fast.next != null) {
        slow = slow.next;          // Move slow by 1
        fast = fast.next.next;     // Move fast by 2
    }

    return slow;  // Slow is at the middle
}
```
*Note: When the list has an even number of nodes, this template returns the second middle node. If you need the first middle node, check `fast.next != null && fast.next.next != null` instead.*

---

## 2. Detecting a Cycle (Is there a loop?)

Sometimes a Linked List is broken and loops back in on itself infinitely. If you just do a normal `while(head != null)`, your code will run forever and time out!

**The Trick & Mathematical Intuition:**
If you put two people on a circular track and one runs twice as fast as the other, the fast runner will eventually "lap" the slow runner and they will collide.

*Why Floyd's cycle detection works:* The standard speed choice is 1 step for the slow pointer and 2 steps for the fast pointer. The relative speed of the fast pointer with respect to the slow pointer is 1 step per iteration. Once both pointers are inside the cycle, the fast pointer gains exactly 1 step on slow every iteration. Since the cycle has finite length L, fast catches up to slow within at most L iterations.

If `slow` and `fast` ever point to the exact same node, a cycle exists!

```text
slow = head
fast = head

// The crucial guard condition: "fast khatam na ho jaye !!!"
WHILE fast != null AND fast.next != null:
    
    slow = slow.next       // 1 step
    fast = fast.next.next  // 2 steps
    
    IF slow == fast:
        return TRUE // CYCLE DETECTED!
        
return FALSE
```
*Note: The `fast != null && fast.next != null` condition is extremely important so your code doesn't crash trying to do `.next.next` on a `null` node.*

**Template (Java):**
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        // If they meet, there's a cycle
        if (slow == fast) {
            return true;
        }
    }

    // Fast reached the end, no cycle
    return false;
}
```

---

## 3. Finding the Starting Point of the Cycle

If you detected a cycle, the next follow-up question is always: "Exactly which node does the cycle start at?"
This involves a bit of heavy mathematics behind the scenes, but the algorithm itself is incredibly simple to write.

**The Trick:**
1. First, detect the cycle using the method above so that `slow == fast` (they collide somewhere inside the loop).
2. Take a brand new pointer `start` and put it at the very beginning of the list (`head`).
3. Now, move BOTH `start` and `fast` (or `slow`) forward by exactly **1 step** at a time.
4. The exact node where they collide again is the starting point of the cycle!

```text
// (Assuming slow and fast already collided inside the cycle)

start = head

WHILE start != fast:
    // Move BOTH pointers by exactly 1 step!
    start = start.next
    fast = fast.next
    
RETURN start // This is the cycle starting node!
```

**Template (Java):**
```java
public ListNode findCycleStart(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    // Phase 1: Detect cycle and find meeting point
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            // Phase 2: Find cycle start
            ListNode pointer = head;
            while (pointer != slow) {
                pointer = pointer.next;
                slow = slow.next;
            }
            return slow;  // Cycle start
        }
    }

    return null;  // No cycle
}
```
