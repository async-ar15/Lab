# 141. Linked List Cycle

Given `head`, the head of a linked list, determine if the linked list has a cycle in it.

There is a cycle in a linked list if there is some node in the list that can be reached again by continuously following the `next` pointer. Internally, `pos` is used to denote the index of the node that tail's `next` pointer is connected to. **Note that `pos` is not passed as a parameter.**

Return `true` *if there is a cycle in the linked list*. Otherwise, return `false`.

### Example 1:
**Input:** head = [3,2,0,-4], pos = 1
**Output:** true
**Explanation:** There is a cycle in the linked list, where the tail connects to the 1st node (0-indexed).

### Example 2:
**Input:** head = [1,2], pos = 0
**Output:** true
**Explanation:** There is a cycle in the linked list, where the tail connects to the 0th node.

### Example 3:
**Input:** head = [1], pos = -1
**Output:** false
**Explanation:** There is no cycle in the linked list.

### Constraints:
- The number of the nodes in the list is in the range `[0, 10^4]`.
- `-10^5 <= Node.val <= 10^5`
- `pos` is `-1` or a **valid index** in the linked-list.

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** If I have nodes `1 -> 2 -> 3`, and `3.next` points back to `2`, it forms a loop. If I try to traverse it (`1 -> 2 -> 3 -> 2 -> 3...`), I will be trapped forever.
- **Clarify edge cases:** What if the linked list is empty (`head == null`) or has only 1 node that doesn't point to itself? In those cases, there is obviously no cycle.
- **Confirm input/output:** 
  - Input: The `head` node of a Linked List.
  - Output: A boolean (`true` if a cycle exists, `false` otherwise).
- **Important keywords:** "cycle", "continuously following the next pointer".
- **Basic understanding:** The problem is asking if the list loops infinitely or if it eventually reaches an end (`null`).

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $10^4$ nodes. An $O(N)$ solution will pass instantly.
- **Space complexity:** A classic follow-up for this problem is to solve it using $O(1)$ memory. 
- **What algorithm to use:** Since we are traversing a Linked List and looking for a cycle, this is the textbook definition of the **Fast and Slow Pointer (Floyd's Cycle-Finding Algorithm)**.

# Solution 

## Brute Force (Using a HashSet)

- **Intuition:** The easiest way to know if we are stuck in a loop is to keep track of every node we visit. We can use a `HashSet` to store the nodes. As we walk through the list, we check if the current node is already in the set. If it is, we found a cycle! If we reach `null`, there is no cycle.
- **Pseudo code:**
```text
SET visited = new HashSet

WHILE head is not null:
    IF visited contains head:
        RETURN true // Cycle detected!
    visited.add(head)
    head = head.next

RETURN false // Reached the end
```
- **Time Complexity:** $O(N)$ because we visit each node at most once.
- **Space Complexity:** $O(N)$ because we are storing every single node in the HashSet! This fails the $O(1)$ space requirement.

## Optimized Code (Fast and Slow Pointers)

- **Intuition:** Instead of remembering every node, we put two runners on a track. The `slow` runner moves 1 step at a time, and the `fast` runner moves 2 steps at a time. 
If the track is a straight line, the `fast` runner will simply reach the end (`null`) and we know there is no cycle.
But if the track is a circle, the `fast` runner will eventually "lap" the `slow` runner from behind. If they ever land on the exact same node (`slow == fast`), we know a cycle exists!
- **Pseudo code:**
```text
1. IF head is null, RETURN false.

2. SET slow = head
3. SET fast = head

4. // Guard condition to prevent NullPointerException
5. WHILE fast is not null AND fast.next is not null:
6.     slow = slow.next       // 1 step
7.     fast = fast.next.next  // 2 steps
8.     
9.     IF slow == fast:
10.        RETURN true // Collision! Cycle detected!
11.        
12. RETURN false // Fast pointer reached the end of the list
```
- **Time Complexity:** $O(N)$. If there is no cycle, `fast` reaches the end in $N/2$ steps. If there is a cycle, they collide in $O(N)$ time.
- **Space Complexity:** $O(1)$ since we are only using two pointers, regardless of the size of the linked list.
- **Solution Code (Java):**
```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
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

# Things told by the instructor

1. Understand the problem
2. Devise a strategy (find edge cases)
3. Breakdown the problem if possible 
4. Write a pseudocode
5. Implement the solution 
6. Testing and debugging 
7. Optimize and review 

---

# Mistakes & Corrections

1. **The Dreaded `NullPointerException`:**
   - *My mistake:* Writing `while (fast != null)` or `while (fast.next != null)` but not both.
   - *Correction:* Because the `fast` pointer takes **2 steps** (`fast.next.next`), you MUST check both `fast != null` AND `fast.next != null` in your `while` loop condition. If `fast.next` is `null`, and you try to call `.next` on it again, your program will instantly crash.

2. **Comparing Values vs Comparing Nodes:**
   - *My mistake:* Writing `if (slow.val == fast.val)`.
   - *Correction:* Two completely different nodes can hold the same value (e.g., a linked list of `1 -> 1 -> 1`). We don't care if the *values* match, we care if they are literally pointing to the exact same *Node object* in memory! Therefore, always check `if (slow == fast)`.
