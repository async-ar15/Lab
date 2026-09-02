# Fast & Slow Pointer Pattern Theory

The Fast and Slow pointer pattern (often called the **Hare and Tortoise Algorithm** or **Floyd's Cycle-Finding Algorithm**) is a clever technique where you use two pointers that travel through a structure (usually a Linked List) at different speeds.

By having one pointer move faster than the other, you can solve three extremely common types of problems:
1. Finding the middle of a Linked List.
2. Detecting if there is a cycle (a loop).
3. Finding the exact starting point of that cycle.

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

---

## 2. Detecting a Cycle (Is there a loop?)

Sometimes a Linked List is broken and loops back in on itself infinitely. If you just do a normal `while(head != null)`, your code will run forever and time out!

**The Trick:**
If you put two people on a circular track and one runs twice as fast as the other, the fast runner will eventually "lap" the slow runner and they will collide.
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
