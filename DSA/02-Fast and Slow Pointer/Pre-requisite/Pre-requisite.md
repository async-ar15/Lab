# Fast and Slow Pointer: Pre-requisites

Before diving into the Fast & Slow pointer pattern, you absolutely must understand the data structure it is most commonly used on: **Linked Lists**. While this pattern can occasionally be used on arrays, 95% of the time, it's a Linked List trick.

## 1. What is a Linked List?
Unlike an Array where all elements are stored right next to each other in memory, a Linked List is made up of individual **Nodes** scattered around in memory. 

Each `Node` has two things:
1. `val`: The actual data/value.
2. `next`: A pointer (or reference) to the *next* Node in the list.

```java
// What a basic Node looks like in code
class ListNode {
    int val;
    ListNode next;
    
    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}
```

## 2. Traversing a Linked List
In an array, if you want the 5th element, you just do `arr[4]`. You can instantly teleport there!
In a Linked List, **you cannot teleport**. To get to the 5th element, you *must* start at the 1st element (`head`), and walk step-by-step using `.next`.

**How to walk one step:**
```java
ListNode temp = head;

// Moving one step forward:
temp = temp.next;
```

**How to walk to the end:**
```java
ListNode temp = head;

while (temp != null) {
    System.out.println(temp.val);
    temp = temp.next; // step forward
}
```

## 3. The Big Danger: NullPointerException
Because Linked Lists rely on pointers, the biggest enemy is reaching the end of the list and trying to go further.
The last node in a list always points to `null` (nothing). 
If `temp` is `null`, and you try to do `temp.next`, your code will completely crash!

**Golden Rule:**
Always check if a node is `null` before trying to read its `.next` property!

## 4. Why Arrays are Sometimes Treated like Linked Lists
Sometimes, a problem will give you an Array, but you actually treat it like a Linked List! 
How? By using the *value* of the array as the *index* for the next jump.
For example, if `nums = [2, 0, 1]`:
- Start at index `0`. Value is `2`. So jump to index `2`.
- At index `2`, Value is `1`. So jump to index `1`.
This is exactly how Linked List jumping works, and it's a very common trick for cycle-detection problems!
