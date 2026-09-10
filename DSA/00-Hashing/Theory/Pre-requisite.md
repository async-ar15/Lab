# Hashing: Pre-requisites

Before diving into Hashing patterns, you must understand the underlying data structures that make it possible: **Hash Maps** and **Hash Sets**. 

> [!NOTE]
> **Prerequisite Reminder**
> While Arrays take $O(N)$ to search for a value, Hash Maps and Hash Sets allow us to search for a value in **$O(1)$ average time**. This magic is what makes the Hashing pattern so powerful.

## 1. How Hashing Works (The Basics)
Behind the scenes, a Hash Map uses a Hash Function. When you give it a key, the hash function mathematically converts that key into an index in an underlying array. 
- **Collisions:** Sometimes two different keys output the same index. Java handles this by storing a Linked List (or a Tree) at that index. Because of this, the *worst-case* time complexity can technically be $O(N)$, but practically, we treat it as **$O(1)$**.

## 2. Hash Maps (Key-Value Pairs)
A Hash Map stores data in `Key -> Value` pairs. You look up the `Key` to find the `Value`. 
- **Keys must be unique.** If you insert a key that already exists, it overwrites the old value.
- **Values can be duplicates.**

| Operation | Time Complexity |
| :--- | :--- |
| **Insert / Put** | $O(1)$ average |
| **Access / Get** | $O(1)$ average |
| **Search / ContainsKey** | $O(1)$ average |
| **Delete / Remove** | $O(1)$ average |
| **Space Complexity** | $O(N)$ (to store N elements) |

### Code Snippet (Java)
```java
import java.util.HashMap;
import java.util.Map;

HashMap<String, Integer> map = new HashMap<>();

// 1. Insert - O(1)
map.put("apple", 5);
map.put("banana", 2);

// 2. Access / Get - O(1)
int apples = map.get("apple"); // returns 5

// 3. Search for Key - O(1)
if (map.containsKey("banana")) {
    System.out.println("We have bananas!");
}

// 4. Update / Overwrite
map.put("apple", 10); // "apple" now maps to 10

// 5. Default Values (Very useful for frequency counting!)
int count = map.getOrDefault("orange", 0); // returns 0 since "orange" isn't in map

// 6. Iterate through a Map - O(N)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}
```

## 3. Hash Sets (Unique Elements Only)
A Hash Set is basically a Hash Map where we only care about the keys. It is used to keep track of a collection of unique items. 

| Operation | Time Complexity |
| :--- | :--- |
| **Insert / Add** | $O(1)$ average |
| **Search / Contains** | $O(1)$ average |
| **Delete / Remove** | $O(1)$ average |

### Code Snippet (Java)
```java
import java.util.HashSet;

HashSet<Integer> set = new HashSet<>();

// 1. Insert - O(1)
set.add(10);
set.add(20);
set.add(10); // Ignored, 10 is already in the set!

// 2. Search - O(1)
if (set.contains(20)) {
    System.out.println("20 is in the set!");
}

// 3. Remove - O(1)
set.remove(10);
```
