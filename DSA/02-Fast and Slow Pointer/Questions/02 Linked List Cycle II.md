# 142. Linked List Cycle II

Given the `head` of a linked list, return *the node where the cycle begins*. If there is no cycle, return `null`.

There is a cycle in a linked list if there is some node in the list that can be reached again by continuously following the `next` pointer. Internally, `pos` is used to denote the index of the node that tail's `next` pointer is connected to (**0-indexed**). It is `-1` if there is no cycle. **Note that `pos` is not passed as a parameter.**

**Do not modify** the linked list.

### Example 1:
**Input:** head = [3,2,0,-4], pos = 1
**Output:** tail connects to node index 1
**Explanation:** There is a cycle in the linked list, where tail connects to the second node.

### Example 2:
**Input:** head = [1,2], pos = 0
**Output:** tail connects to node index 0
**Explanation:** There is a cycle in the linked list, where tail connects to the first node.

### Example 3:
**Input:** head = [1], pos = -1
**Output:** no cycle
**Explanation:** There is no cycle in the linked list.

### Constraints:
- The number of the nodes in the list is in the range `[0, 10^4]`.
- `-10^5 <= Node.val <= 10^5`
- `pos` is `-1` or a **valid index** in the linked-list.

**Follow up:** Can you solve it using `O(1)` (i.e. constant) memory?

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** If `1 -> 2 -> 3 -> 4` and `4` points back to `2`, the cycle begins at node `2`. We need to return the actual `Node` object for `2`, not the index, and not just `true/false`.
- **Clarify edge cases:** If `head == null` or there is only 1 node with no cycle, return `null`.
- **Confirm input/output:** 
  - Input: The `head` node of the Linked List.
  - Output: The specific `Node` where the cycle begins (or `null` if no cycle exists).
- **Important keywords:** "node where cycle begins", "do not modify".
- **Basic understanding:** This is a direct sequel to the standard Cycle Detection problem. Once we find out a cycle exists, we have to do extra work to backtrack and find the exact starting point.

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $10^4$ nodes means an $O(N)$ solution will pass instantly.
- **Space complexity:** The follow up specifically asks for $O(1)$ memory, which means we cannot use external data structures to track our path.
- **What algorithm to use:** $O(1)$ Space + Finding a cycle starting point = **Floyd's Cycle-Finding Algorithm** (Fast and Slow Pointers with the reset trick).

# Solution 

## Brute Force (Using a HashSet)

- **Intuition:** The easiest way is to walk through the linked list and drop every node we visit into a `HashSet`. If we ever visit a node that is *already* in the set, that node MUST be the start of the cycle!
- **Pseudo code:**
```text
SET visited = new HashSet

WHILE head is not null:
    IF visited contains head:
        RETURN head // We found the exact node where the cycle starts!
    visited.add(head)
    head = head.next

RETURN null // No cycle found
```
- **Time Complexity:** $O(N)$ since we visit each node at most once.
- **Space Complexity:** $O(N)$ because we are storing every single node inside the HashSet. This fails the follow-up challenge!

## Optimized Code (Floyd's Cycle-Finding Algorithm)

- **Intuition:** We use the two-pointer math trick. 
  1. First, we send `slow` (1 step) and `fast` (2 steps) down the list until they collide. This proves a cycle exists.
  2. The moment they collide, we take one of the pointers (let's say `slow`) and teleport it all the way back to the `head` of the list.
  3. Now, we move BOTH `slow` and `fast` forward at the exact same speed (**1 step at a time**).
  4. The exact node where they collide for the second time is mathematically guaranteed to be the start of the cycle!
- **Pseudo code:**
```text
1. IF head is null, RETURN null

2. SET slow = head
3. SET fast = head

4. // Step 1: Find the collision point
5. WHILE fast is not null AND fast.next is not null:
6.     slow = slow.next
7.     fast = fast.next.next
8.     
9.     IF slow == fast:
10.        // COLLISION! We found a cycle. 
11.        // Step 2: Teleport one pointer back to the start
12.        slow = head
13.        
14.        // Step 3: Move both at the same speed until they meet again
15.        WHILE slow != fast:
16.            slow = slow.next
17.            fast = fast.next
18.            
19.        // Step 4: The meeting point is the start of the cycle!
20.        RETURN slow 

21. RETURN null // Fast hit the end of the list, no cycle.
```
- **Time Complexity:** $O(N)$.
- **Space Complexity:** $O(1)$ because we only use two pointers.

- **Solution Code (Java):**
```java
// Write your Java code here based on the pseudo-code!
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

1. **Storing Values instead of Nodes in the HashSet:**
   - *My mistake:* Thinking we could just store the `node.val` in the HashSet to detect a cycle.
   - *Correction:* A linked list can have multiple completely different nodes that happen to have the same value (e.g., `2 -> 4 -> 2 -> 4`). If you store values, you will falsely trigger a cycle detection on the second `2`! You must store the actual `Node` object itself in the HashSet because memory addresses are universally unique.

2. **The Algorithm Name Mix-up:**
   - *My mistake:* Calling it the "Floyd-Warshall algorithm".
   - *Correction:* It is **Floyd's Cycle-Finding Algorithm**. Floyd-Warshall is a completely different (and much more complex) algorithm used for finding the shortest path between all pairs of nodes in a weighted graph!
