### related to the constraints 
- the  constraints numbers.length <= 3 * 10^4 is the most important one in the most important one.
- if an array size is around 10^4 to 10^5, 
- an O(n^2) algo (like our brute force ) will do roughly (30,000)^2 = 900,000,000 operations 
- that is 10^9 operation but standard computer does only 10^8 operations 
- so it will give the TLE 
- so we must find an approach that takes 0(n) or 0(nlogn) time 
- if the problem specifies constant extra space which means space complexity  must be strictly 0(1)

**TL;DR:** 
- **Time limit:** Array size $10^4$ to $10^5$ $\rightarrow$ $O(n^2)$ will give TLE. You must use $O(n)$ or $O(n \log n)$.
- **Space limit:** "Constant extra space" $\rightarrow$ Strictly $O(1)$ space complexity.

### Constraint Cheat Codes (The "Trick")
When an interview problem restricts you to **Time = $O(n)$** and **Space = $O(1)$**, your brain should immediately jump to these specific patterns:
1. **Two Pointers:** (Moving from opposite ends or same direction). *The holy grail for sorted arrays!*
2. **Fast & Slow Pointers:** (Tortoise and Hare). Usually for Linked Lists or finding cycles.
3. **Sliding Window:** For finding contiguous subarrays (only if you don't need a hash map to track things).
4. **Greedy / One-Pass:** Looping once while keeping track of a running max/min (e.g., Kadane's Algorithm for max subarray).
5. **Bit Manipulation:** Using XOR to find single/missing numbers.

When you see **Time = $O(n \log n)$** and **Space = $O(1)$**, immediately think of:
1. **Sorting the array first:** (Though keep in mind some languages use $O(\log n)$ or $O(n)$ space under the hood for sorting).
2. **Binary Search in a Loop:** Iterating through the array $O(n)$ and doing an iterative binary search $O(\log n)$ for each element.