# 876. Middle of the Linked List

Given the `head` of a singly linked list, return *the middle node of the linked list*.

If there are two middle nodes, return **the second middle** node.

### Example 1:
**Input:** head = [1,2,3,4,5]
**Output:** [3,4,5]
**Explanation:** The middle node of the list is node 3.

### Example 2:
**Input:** head = [1,2,3,4,5,6]
**Output:** [4,5,6]
**Explanation:** Since the list has two middle nodes with values 3 and 4, we return the second one.

### Constraints:
- The number of nodes in the list is in the range `[1, 100]`.
- `1 <= Node.val <= 100`

---

# Understanding the Question 

Breaking down the problem:
- **Draw examples:** 
  - Odd number of nodes: `1 -> 2 -> 3 -> 4 -> 5`. The exact middle is `3`.
  - Even number of nodes: `1 -> 2 -> 3 -> 4 -> 5 -> 6`. There are two middles (`3` and `4`), but the question explicitly says to return the *second* middle node, which is `4`.
- **Confirm input/output:** 
  - Input: The `head` Node of the Linked List.
  - Output: The actual `Node` object representing the middle (not just the integer value).
- **Important keywords:** "middle node", "second middle node".

# Understanding the Constraints

What the constraints are secretly telling me:
- **Time complexity:** $100$ nodes is incredibly small. Any $O(N)$ solution will run practically instantly.
- **Space complexity:** We shouldn't need any extra space. $O(1)$ space is the standard expectation for basic Linked List traversal.
- **Edge cases:** The minimum length is `1`. This means we don't have to worry about `head == null`. If there is only 1 node, it is the middle node.

# Solution 

## Brute Force (Two Passes / Counting)

- **Intuition:** Since a Linked List doesn't have indices like an array, we don't know how long it is. So, let's just walk through the entire list once to count how many nodes there are. Then, we divide the count by 2 to find the exact index of the middle. Finally, we walk through the list a second time, stopping at the middle index!
- **Pseudo code:**
```text
// Pass 1: Count nodes
SET count = 0
SET current = head
WHILE current is not null:
    count = count + 1
    current = current.next

// Find the middle index
SET middleIndex = count / 2

// Pass 2: Traverse to the middle
SET current = head
FOR i from 0 to middleIndex - 1:
    current = current.next

RETURN current
```
- **Time Complexity:** $O(N) + O(N/2) = O(N)$. We pass through the list 1.5 times.
- **Space Complexity:** $O(1)$. We only use a couple of variables to keep track of counting.

## Optimized Code (Fast & Slow Pointers / Tortoise and Hare)

- **Intuition:** Instead of doing 1.5 passes, we can do it in exactly 1 pass using the Fast & Slow pointer pattern! 
  If two people are running a race, and Runner B runs exactly TWICE as fast as Runner A... by the time Runner B crosses the finish line, Runner A will be at exactly the **halfway point** (the middle)!
  - `slow` takes 1 step at a time.
  - `fast` takes 2 steps at a time.
  When `fast` hits the end of the list, `slow` is guaranteed to be standing on the middle node.
- **Pseudo code:**
```text
SET slow = head
SET fast = head

WHILE fast is not null AND fast.next is not null:
    slow = slow.next          // 1 step
    fast = fast.next.next     // 2 steps

RETURN slow
```
- **Time Complexity:** $O(N/2) = O(N)$. We only traverse half the list because `fast` reaches the end twice as quickly.
- **Space Complexity:** $O(1)$. We only use two pointers.

- **Solution Code (Java):**
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

1. **NullPointerException Danger:**
   - *Common Mistake:* Writing the while loop as `while (fast != null)`.
   - *Correction:* If `fast` is standing on the very last node (meaning `fast.next` is null), taking a 2-step jump via `fast.next.next` will crash your code with a `NullPointerException` because you are asking for the `.next` of a null object! You **MUST** always check `fast.next != null` before attempting a 2-step jump.

2. **The "Second Middle Node" Edge Case:**
   - *Common Mistake:* Trying to do complex math or weird starting positions to make sure it returns the *second* middle node for even-length lists.
   - *Correction:* The standard Fast & Slow algorithm naturally handles this! By starting both pointers exactly at `head`, the math works out perfectly so that when `fast` falls off the end of an even-length list, `slow` automatically lands on the second middle node. No extra math required!
