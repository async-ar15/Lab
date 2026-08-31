# 01 Arrays

An array is a collection of items stored at contiguous (side-by-side) memory locations. Because they are stored next to each other, the computer can instantly calculate where any item is if it knows the starting address.

Key Characteristics:

- Fixed Size (usually): In lower-level languages like C/Java, arrays have a fixed size. If you need a bigger one, you have to create a new one and copy the elements.
- Dynamic Arrays: Languages like Python (Lists), JavaScript, or C++ (Vectors) handle resizing automatically behind the scenes, but the underlying concept is the same.

## Table of Content

| Operation | Time Complexity |
| :--- | :--- |
| **Access (Index)** | $O(1)$ |
| **Search (Value)** | $O(N)$ |
| **Insert/Delete (End)** | $O(1)$ |
| **Insert/Delete (Middle/Start)** | $O(N)$ |
| **Space Complexity** | $O(N)$ (to store N elements) |



## Code snippets 

In Java, you have fixed-size primitive arrays and dynamic ArrayLists. You will use both frequently depending on the problem.

```
// --- PRIMITIVE ARRAYS (Fixed Size) ---
int[] arr = {10, 20, 30, 40, 50};

// 1. Access & Modify by Index - O(1)
int firstElement = arr[0];      // 10
arr[4] = 60;                    // Change 50 to 60

// 2. Iterate / Traverse - O(N)
for (int i = 0; i < arr.length; i++) {
    System.out.println("Index " + i + ": " + arr[i]);
}
// Or using an enhanced for-loop
for (int num : arr) {
    System.out.println(num);
}

// --- ARRAYLISTS (Dynamic Size) ---
import java.util.ArrayList;
import java.util.List;

List<Integer> list = new ArrayList<>(List.of(10, 20, 30));

// 1. Access by Index - O(1)
int num = list.get(0);          // 10

// 2. Insert at End - O(1) average
list.add(40);                   // [10, 20, 30, 40]

// 3. Insert in Middle/Start - O(N) (Requires shifting elements)
list.add(1, 15);                // [10, 15, 20, 30, 40]

// 4. Delete from Middle/Start - O(N) (Requires shifting elements)
list.remove(1);                 // Removes element at index 1 (the 15)

// 5. Delete from End - O(1)
list.remove(list.size() - 1);   // Removes 40


```


# 2 Strings 

2. Strings
A string is essentially an array of characters. The concepts are almost identical to arrays, but with one massive catch in many programming languages: Immutability.

Key Characteristics:

- Immutable (in Java, Python, C#, JS): Once a string is created, you cannot change it. If you do str = str + "a", it doesn't just add "a" to the end. It creates a brand-new string in memory, copies the old characters, and adds "a".

- Mutable (in C, C++): Strings can be changed in-place like normal arrays.

Operation	Time Complexity
Access (Index)	$O(1)$
Search (Value)	$O(N)$
Concatenation / Append	$O(N)$ (due to immutability)
Substring (length K)	$O(K)$
Space Complexity	$O(N)$


## Code snippets 

- In java string are mutuable 

```
String s = "hello";

// 1. Access by Index - O(1)
char c = s.charAt(1);           // 'e'

// 2. Substring - O(K) where K is length of the new string
// (Start index is inclusive, end index is exclusive)
String sub = s.substring(1, 4); // "ell"

// 3. Convert to Char Array (Very common in Two Pointer problems!) - O(N)
char[] charArray = s.toCharArray();
charArray[0] = 'H';             // Now you can swap/modify characters in-place!

// --- STRINGBUILDER (For efficiently building/modifying strings) ---

// 4. The BAD way to build strings in a loop - O(N^2)
String badString = "";
for (char ch : new char[]{'a', 'b', 'c', 'd'}) {
    badString += ch;            // Creates a brand new String object every time!
}

// 5. The GOOD way - O(N)
StringBuilder sb = new StringBuilder();
for (char ch : new char[]{'a', 'b', 'c', 'd'}) {
    sb.append(ch);              // O(1) append to an internal array
}
String goodString = sb.toString(); // O(N) to convert at the end


```

# 3 Linkedlist 

$O(N)$
3. Linked Lists
A Linked List is a collection of elements (called Nodes) that are not stored in contiguous memory. Instead, each node contains the data AND a "pointer" (a reference) to the exact memory address of the next node in the sequence.

Types:

Singly Linked List: Nodes point only to the next node (A -> B -> C).
Doubly Linked List: Nodes point to the next AND the previous node (A <-> B <-> C).

## table of content

Operation	Time Complexity
Access (Index)	$O(N)$
Search (Value)	$O(N)$
Insert/Delete (at Head)	$O(1)$
Insert/Delete (Middle/End)	$O(N)$ (due to finding the spot)
Space Complexity	$O(N)$ (uses slightly more memory than arrays due to pointers)

## code snippets


```

// 1. Defining the Node (Standard LeetCode definition)
class ListNode {
    int val;
    ListNode next;
    
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}

public class LinkedListOperations {

    // 2. Initialize a Linked List: 10 -> 20 -> 30
    public static void main(String[] args) {
        ListNode head = new ListNode(10);
        head.next = new ListNode(20);
        head.next.next = new ListNode(30);
    }

    // 3. Traverse / Print the List - O(N)
    public void printList(ListNode head) {
        ListNode curr = head;
        while (curr != null) {
            System.out.print(curr.val + " -> ");
            curr = curr.next;
        }
        System.out.println("null");
    }

    // 4. Insert at the Head - O(1)
    public ListNode insertAtHead(ListNode head, int value) {
        ListNode newNode = new ListNode(value);
        newNode.next = head;
        return newNode;         // This is the new head!
    }

    // 5. Delete a Node (given the head and a target value) - O(N)
    public ListNode deleteNode(ListNode head, int target) {
        // Edge case: Empty list
        if (head == null) return null;
        
        // Edge case: Target is the head itself
        if (head.val == target) {
            return head.next;   // Just move the head over
        }
        
        // Standard case: Traverse to find the node BEFORE the target
        ListNode curr = head;
        while (curr.next != null && curr.next.val != target) {
            curr = curr.next;
        }
        
        // If we found the target (curr.next is not null), skip it
        if (curr.next != null) {
            curr.next = curr.next.next;
        }
        
        return head;
    }
}



```
