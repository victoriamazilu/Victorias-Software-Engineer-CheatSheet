# LeetCode Patterns and Strategies

Some strategies, patterns, etc. to keep in mind when solving.

## 1. Sliding Window

The Sliding Window pattern is used to perform operations on a specific window size of a given array or linked list, such as finding the longest subarray with a specific condition. The window slides from the start to the end of the data structure, dynamically adjusting its size based on the problem requirements.

## 2. Two Pointers

The Two Pointers technique involves using two pointers to iterate through a data structure in tandem. This is especially useful for problems that require comparing elements within a sorted array or linked list, such as finding pairs that meet a certain condition. It optimizes space and time complexity by reducing the need for nested loops.

## 3. Fast and Slow Pointers

Also known as the Hare and Tortoise algorithm, this approach uses two pointers moving at different speeds to detect cycles in linked lists or arrays. It's particularly effective for problems involving cyclic structures, ensuring the fast pointer eventually catches up with the slow pointer if a cycle exists.

## 4. Hash Map

Hash Maps are used for problems that require constant-time lookups. They store key-value pairs and are useful for problems involving counting frequencies, storing and retrieving elements quickly, and implementing dictionaries or caches.

## 5. Stack

Stacks are LIFO (Last In, First Out) data structures. They are useful for problems involving recursive tree or graph traversals, evaluating expressions, or implementing undo functionalities. Stacks are used to keep track of function calls or to reverse data.

## 6. Queue

Queues are FIFO (First In, First Out) data structures. They are ideal for problems involving level-order tree traversal (BFS), scheduling tasks, or managing resources in a fair order. Queues help in processing elements in the order they are added.

## 7. Merge Intervals

The Merge Intervals pattern efficiently handles problems involving overlapping intervals. It helps in merging overlapping intervals or finding intersections by understanding the different ways intervals can relate to each other.

## 8. Cyclic Sort

Cyclic Sort is ideal for problems involving arrays with elements in a specific range. It sorts elements by swapping them to their correct positions, minimizing time complexity compared to traditional sorting algorithms.

## 9. In-place Reversal of Linked List

This pattern is used to reverse the links between nodes of a linked list in-place, without using extra memory. It iterates through the list, reversing one node at a time, using two pointers to keep track of the current and previous nodes.

## 10. Tree BFS

Tree Breadth First Search (BFS) traverses a tree level by level, using a queue to manage the nodes. It's useful for problems requiring level-order traversal, such as finding the shortest path in a tree.

## 11. Tree DFS

Tree Depth First Search (DFS) uses recursion or a stack to traverse a tree. Depending on when nodes are processed, it can be pre-order, in-order, or post-order traversal. It's suited for problems requiring deep traversal of trees.

## 12. Two Heaps

Two Heaps is a pattern for problems where elements need to be divided into two parts. It uses a Min Heap for one part and a Max Heap for the other, efficiently finding the median or other statistics of the combined data.

## 13. Subsets

The Subsets pattern handles problems involving permutations and combinations. It uses a BFS approach to generate all possible subsets of a given set, iterating through elements and adding them to existing subsets.

## 14. Modified Binary Search

Modified Binary Search is used for searching in sorted arrays, linked lists, or matrices. It efficiently finds elements by repeatedly dividing the search interval in half, reducing time complexity compared to linear search.

## 15. Top K Elements

This pattern deals with finding the top, smallest, or most frequent 'K' elements in a dataset. It uses a heap to maintain 'K' elements, iterating through the data to update the heap as needed.

## 16. K-way Merge

K-way Merge is for problems involving multiple sorted arrays. It uses a heap to perform a sorted traversal of all elements by merging arrays efficiently, maintaining overall sorted order.

## 17. Topological Sort

Topological Sort finds a linear ordering of elements with dependencies. It builds a graph representation using adjacency lists and in-degrees, processing nodes with zero in-degrees first and iterating until all nodes are sorted.
