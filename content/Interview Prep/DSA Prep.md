---
{"publish":true,"created":"2026-02-14T09:10:40.663Z","modified":"2026-08-17T03:03:10.088Z"}
---

# Story Based questions

## 1. Greedy / Sorting “deadline” stories

1. **Eliminate Maximum Number of Monsters / Airplanes variant** (Greedy + sort by arrival time)\
   You defend a city from monsters/planes; each has a starting distance and speed, you shoot once per minute, and lose when any reaches the city. Max monsters/planes you can eliminate.

2. **Meeting Rooms II – “Conference scheduling”** (Intervals + min-heap)\
   You’re given start/end times of meetings and must find the minimum number of rooms needed so no meetings overlap in a room.

3. **Task Scheduler – “CPU with cooldown”** (Greedy + counting)\
   Tasks are labeled with letters; the CPU needs a cooldown between identical tasks. Find least time units to finish all tasks with idle slots allowed.

4. **Non-overlapping Intervals – “Erase minimum number of activities”** (Greedy by end time)\
   Each activity has a time interval; you must remove as few as possible so the remaining don’t overlap.

5. **Jump Game / Min Jumps – “Frog crossing stones”** (Greedy / DP)\
   A frog at index 0 can jump up to nums\[i] steps from stone i; determine if it can reach the last stone or minimum jumps needed.

---

## 2. Monotonic stack / “next greater / visible buildings” stories

6. **Daily Temperatures – “Hotter day forecast”** (Monotonic stack)\
   Given daily temperatures, for each day find how many days until a warmer temperature; 0 if none.

7. **Largest Rectangle in Histogram – “Largest billboard from buildings”** (Monotonic stack)\
   Given building heights, find the largest rectangular billboard area that can be formed by contiguous buildings.[](https://abaj.ai/projects/ccs/dsa/leetcode/problem-list/)​

8. **Trapping Rain Water – “Water between bars”** (Two pointers / stack)\
   Given an elevation map, compute how much water can be trapped after raining.

9. **Buildings With an Ocean View – “Visible buildings”** (Monotonic stack / scan from right)\
   A row of buildings faces the ocean; return indices of buildings that can see the ocean (no taller building to the right).

10. **Next Greater Element (circular) – “Warmer/greater stock prices in a loop”** (Monotonic stack)\
    In a circular array, for each element find the next strictly greater element scanning forward, wrapping at the end.

---

## 3. Sliding window “string/array game” stories

11. **Longest Substring Without Repeating Characters – “Unique characters in a signal”** (Sliding window)\
    A signal is a string; find length of the longest substring without repeating characters.

12. **Longest Substring with At Most K Distinct Characters – “User session with at most K apps”** (Sliding window + hashmap)\
    You want the longest continuous period where the user has used at most K distinct apps (characters).

13. **Minimum Window Substring – “Smallest snippet containing all keywords”** (Sliding window + frequency map)\
    Given a document and a set of keywords, find the smallest contiguous snippet that contains all keywords.

14. **Max Consecutive Ones III – “Longest streak with at most K repairs”** (Sliding window)\
    In a binary array of working/broken machines, flip at most K broken ones to working to get the longest all-working streak.

15. **Fruits into Baskets – “Pick at most two fruit types”** (Sliding window)\
    You move along trees; you can carry at most two fruit types; find the max fruits you can pick in a row.[](https://blog.algomaster.io/p/15-leetcode-patterns)​

---

## 4. Heap / priority-based stories

16. **K Closest Points to Origin – “Closest delivery locations”** (Min/Max-heap)\
    Given coordinates of houses, return k closest to warehouse at origin based on Euclidean distance.

17. **Top K Frequent Elements – “Most frequent search terms”** (Heap / bucket sort)\
    Log of user queries; find the k most frequent queries.

18. **Merge K Sorted Lists – “Merge K sorted log streams”** (Min-heap over heads)\
    Merge k sorted logs from different servers into a single time-sorted log.

19. **Task with Deadlines and Profits – “Maximize jobs done before deadlines”** (Greedy + heap)\
    Each job has a deadline and profit; choose subset to maximize profit with at most one job at a time.

20. **Meeting Scheduler – “Earliest meeting slot between two people”** (Heap / two pointers on sorted intervals)\
    Two users have available time slots; find the earliest common slot of at least given duration.

---

## 5. Prefix sum / interval / counting stories

21. **Subarray Sum Equals K – “Number of balanced days”** (Prefix sum + hashmap)\
    Given daily net gain/loss, count how many continuous periods sum to exactly K.

22. **Range Sum Query – “Bank account statement queries”** (Prefix sum)\
    Preprocess balances so multiple queries asking sum between days i and j are fast.[](https://abaj.ai/projects/ccs/dsa/leetcode/problem-list/)​

23. **Car Pooling – “Capacity along a route”** (Difference array / sweep line)\
    Trips specify passengers, pickup and drop locations; determine if car capacity is never exceeded.[](https://blog.algomaster.io/p/15-leetcode-patterns)​

24. **Corporate Flight Bookings – “Bookings affecting flight seats”** (Difference array)\
    Each booking adds seats to a range of flights; compute seats booked per flight.[](https://blog.algomaster.io/p/15-leetcode-patterns)​

25. **Number of Subarrays with Bounded Maximum – “Segments with acceptable max risk”** (Counting + two pointers)\
    Count subarrays whose maximum value lies in \[L, R], seen as time intervals with acceptable risk.

---

## 6. Graph / BFS-DFS stories

26. **Number of Islands – “Count groups of lands”** (DFS/BFS on grid)\
    You’re given a map of land/water tiles; count distinct islands.

27. **Rotting Oranges – “Spreading infection”** (Multi-source BFS)\
    Rotten oranges infect adjacent fresh ones each minute; find minutes until all rot or return -1.[](https://blog.algomaster.io/p/15-leetcode-patterns)​

28. **Word Ladder – “Shortest transformation sequence”** (BFS on implicit graph)\
    Change one character at a time, intermediate words must be in dictionary; find shortest transformation length.

29. **Course Schedule – “Can you finish all courses?”** (Topological sort)\
    Courses have prerequisites; determine if you can complete all or detect cycle.

30. **Open the Lock – “Shortest moves to unlock”** (BFS + visited set)\
    A lock has 4 wheels (0000 to 9999), some forbidden states, a target; find minimum moves.

---

## 7. “Design / streaming” stories

31. **LRU Cache – “Most recently used pages”** (HashMap + doubly linked list)\
    Design a cache that evicts least recently used key on capacity overflow.[](https://abaj.ai/projects/ccs/dsa/leetcode/problem-list/)​

32. **Min Stack – “Stack with getMin in O(1)”** (Two stacks or encoded values)\
    Stack that supports push, pop, top, and retrieving minimum in constant time.

33. **Median of Data Stream – “Live median of numbers”** (Two heaps)\
    Numbers arrive one by one; support addNum and findMedian efficiently.[](https://abaj.ai/projects/ccs/dsa/leetcode/problem-list/)​

34. **Insert Delete GetRandom O(1) – “Randomized set of items”** (HashMap + array)\
    Design a set that supports insert, delete, and getRandom in expected O(1).

35. **Time-based Key-Value Store – “Versioned config store”** (Map + binary search per key)\
    Set(key, value, timestamp) and get(key, timestamp) return value at latest time ≤ timestamp.

---

## How to practice for Agoda with these

For each problem above:

1. **Hide the title and tags.** Just read the story and ask: “What is the underlying DS/pattern?”

2. **First step:** classify it (e.g., monotonic stack / greedy / two heaps / sliding window).

3. **Second step:** outline operations (sort, scan once, two pointers, heap ops, etc.).

4. **Third step:** implement with attention to constraints and edge-cases.

If you want, next step I can:

- Map each of these to **exact LeetCode IDs** you can queue in a list, and

- Give you a **checklist** like “for every story at Agoda, first ask yourself these 5 questions” so you have a routine during the round.

# Backtracking vs DP vs Recursion

| **Feature**                  | **Backtracking**                                           | **Dynamic Programming**                                     |
| ---------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------- |
| **Primary Goal**             | Generate patterns / Find valid paths.                      | Minimize/Maximize cost / Count ways.                        |
| **Logic**                    | **Brute Force with Undo**.                                 | **Brute Force with Memory**.                                |
| **State**                    | **Path Dependent.** (Who is my parent? What did I choose?) | **State Dependent.** (Where am I? What resources are left?) |
| **Overlapping Subproblems?** | No (or we ignore them).                                    | **YES.** This is the _core requirement_ for DP.             |
| **Complexity**               | **Exponential** ($O(2^N)$, $O(N!)$).                       | **Polynomial** ($O(N^2)$, $O(N^3)$).                        |

## Intuition :

When you see a new problem, follow this flowchart to decide which one to use.

#### Step 1: Draw the Recursive Tree

Start with standard recursion. Imagine the decision tree.

- _Question:_ "Can I solve this by making a choice and solving the rest?"

- _If Yes:_ You have a Recursive solution.

#### Step 2: Check for Overlap ( The "DP Test" )

Look at different branches of your tree. Do they ever reach the **exact same state** with the **exact same remaining parameters**?

- _Example (Fibonacci):_ `fib(5)` calls `fib(4)` and `fib(3)`. `fib(4)` _also_ calls `fib(3)`.

- **Result:** The state `fib(3)` appears twice.

- **Conclusion:** This is **Dynamic Programming**. Use a cache (Memoization) to store `fib(3)`.

#### Step 3: Check the Dependency ( The "Backtracking Test" )

- **Choose:** Make a choice (add an item).

- **Explore:** Call function again (recurse).

- **Un-Choose (Backtrack):** **Undo** the choice (remove the item) so you can try a different choice in the next iteration.

Does the solution to the sub-problem depend on the _items you already picked_?

- _Example (N-Queens):_ You place a Queen at `(0,0)`. Now you try to place one at `(1,0)`. You _can't_, because the first Queen attacks it.

- **Result:** The validity of the current step depends entirely on the history of previous steps.

- **Conclusion:** This is **Backtracking**. You cannot memoize "Row 2" because "Row 2" changes depending on where you placed the Queen in Row 1.

## Example Problems

#### Level 1

[LeetCode #104: Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
[LeetCode #46: Permutations](https://leetcode.com/problems/permutations/)
[LeetCode #70: Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
