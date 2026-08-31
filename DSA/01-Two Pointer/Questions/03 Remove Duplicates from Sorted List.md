# 83. Remove Duplicates from Sorted List

Given the `head` of a sorted linked list, *delete all duplicates such that each element appears only once*. Return *the linked list **sorted** as well*.

### Example 1:
**Input:** head = [1, 1, 2]
**Output:** [1, 2]

### Example 2:
**Input:** head = [1, 1, 2, 3, 3]
**Output:** [1, 2, 3]

### Constraints:
- The number of nodes in the list is in the range `[0, 300]`.
- `-100 <= Node.val <= 100`
- The list is guaranteed to be **sorted** in ascending order.

---

# Understanding the Question 

Breaking it down:
- **Draw examples:** `1 -> 1 -> 2` becomes `1 -> 2`. If I have `1 -> 1 -> 1 -> 2 -> 3 -> 3`, it becomes `1 -> 2 -> 3`.
- **Clarify edge cases:** The constraints say `0` nodes is possible! So I have to handle an empty list `head == null` right away to prevent a `NullPointerException`. Also, a list with just 1 node should return as-is.
- **Confirm input/output:** 
  - Input: `head` node of a LinkedList.
  - Output: `head` node of the modified LinkedList.
- **Important keywords:** "sorted in ascending order", "linked list".
- **Basic understanding:** The problem is just about traversing the linked list. Since it's sorted, any duplicates will be right next to each other. I just need to snip out the extra ones by rewiring the `next` pointers.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** Max nodes `300`. This is tiny! I could practically run a crazy $O(n^3)$ algorithm and it would probably still pass. But since it's a simple list traversal, an $O(n)$ solution is the most natural way to do it.
- **Space complexity:** I don't need any extra space. I can just manipulate the existing nodes in memory, which gives me strictly $O(1)$ space.
- **What algorithm to use:** Fast & Slow pointers / simple pointer traversal. 

# Solution 

## Brute Force (Hash Set)

- **Intuition:** If the list *wasn't* sorted, the easiest way to remove duplicates would be to use a `HashSet`. I would traverse the list, and if a number isn't in the set, add it. If it is in the set, remove that node.
- **Pseudo code:**
```text
SET seen = new HashSet
SET current = head
SET prev = null

WHILE current is not null:
    IF seen contains current.val:
        prev.next = current.next (skip it)
    ELSE:
        seen.add(current.val)
        prev = current
    
    current = current.next
```
- **Time Complexity:** $O(n)$ because we traverse the list once.
- **Space Complexity:** $O(n)$ because the HashSet could store up to $n$ unique elements.
- **Thoughts:** We can do better on space! Since the list is *already sorted*, we don't need the HashSet at all.

## Optimized Code (Single Pointer Traversal)

- **Intuition:** Since the list is sorted, duplicates are glued together. I can just stand at a node (`current`), and peek at the *next* node (`current.next`). If they have the exact same value, I completely bypass the next node by linking `current.next` to `current.next.next`! But wait—I can't move my `current` pointer forward just yet, because the *new* next node might ALSO be a duplicate! I only move forward when I'm sure the next node is different.
- **Pseudo code:**
```text
1. IF head is null, RETURN head (edge case!)
2. SET current = head

3. WHILE current is not null AND current.next is not null:
4.     IF current.val == current.next.val:
5.         current.next = current.next.next // Snip it out!
6.     ELSE:
7.         current = current.next // Safe to move forward
       
8. RETURN head
```
- **Time Complexity:** $O(n)$ because in the worst case we process each node exactly once. (Even when we skip nodes, we reduce the total nodes to process).
- **Space Complexity:** $O(1)$ because we only use one extra pointer variable (`current`).
- **Solution Code (Java):**
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

# Mistakes & Corrections

Here are the classic mistakes I always make with Linked Lists:

1. **NullPointerException on Empty Lists**: 
   - *My mistake:* Jumping straight into `ListNode current = head; while(current.next != null)` without checking if `head` itself is null first! 
   - *Correction:* Always remember that `Constraints: nodes range [0, 300]` means 0 nodes is possible. Start with `if (head == null) return head;`.

2. **Moving the pointer too early**:
   - *My mistake:* Writing `current.next = current.next.next; current = current.next;` 
   - *Correction:* If the list is `1 -> 1 -> 1`, skipping the first duplicate makes it `1 -> 1`. If I move `current` forward immediately, I'll step onto the second `1` and miss the chance to delete it. I must only `current = current.next` in the `else` block!

3. **Checking `current.next.val` when `current.next` is null**:
   - *My mistake:* Writing a `while (current != null)` loop and inside it doing `if (current.val == current.next.val)`. If `current` is the very last node, `current.next` is `null`, and asking for `.val` throws a `NullPointerException`.
   - *Correction:* The loop condition *must* be `while (current != null && current.next != null)`.

