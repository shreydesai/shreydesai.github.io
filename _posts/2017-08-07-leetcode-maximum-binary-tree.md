---
layout: post
title: "Leetcode - Maximum Binary Tree"
date: 2017-08-07
comments: true
---

This week, I'm going to cover the [Maximum Binary Tree](https://leetcode.com/contest/leetcode-weekly-contest-44/problems/maximum-binary-tree/) problem. It's an interesting tree problem that combines multiple concepts such as searching, traversals, and recursion.

# Problem

Given an integer array with no duplicates, a maximum tree building on this array is defined as follows:

1. The root is the maximum number in the array.
2. The left subtree is the maximum tree constructed from the left part subarray divided by the maximum number.
3. The right subtree is the maximum tree constructed from the right part subarray divided by the maximum number.

Construct the maximum tree by the given array and output the root node of this tree.

For example, the array `[3, 2, 1, 6, 0, 5]` would give us the following tree:

```text
      6
    /   \
   3     5
    \   /
     2 0
      \
       1
```

# Solution

When reading the problem statement, the first thing that should go through your mind is **recursion**. Once steps 1-3 are completed for the root node, the steps can also repeated for the left and right children, assuming they exist. Therefore, the key to solving the problem is solving it for once case correctly so that it can be recursively applied to other cases.

Let's take a look at the first couple steps of the algorithm. The maximum number in the array is `6`, which exists at index `3`. The maximum number in the left subarray (from indices `0 -> 2`) is `3`. The maximum number in the right subarray (from indices `4 -> 5`) is `5`. The root node is `6` and the left and right children are `3` and `5` respectively. We can repeat the same algorithm on the left and right subarrays until the array is empty.

The process of visiting a root node then recursing on its left and right subtrees should seem familiar. This is a [DFS pre-order](https://en.wikipedia.org/wiki/Tree_traversal#Pre-order) tree traversal. The only difference is that we have to build our tree as we traverse it; this shouldn't be a problem because the values are supplied by the maximum number in each recursive step.

Therefore, we are left with the following algorithm:

- Check if `nums` is empty. If it is, return `None`. This will presumably form the leaves of our tree.
- Find the index of the maximum number. Let's say it's at index `i`.
- Create a node whose internal element is `nums[i]`.
- Recurse on the left subtree with `nums[:i]` and the right subtree with `nums[i + 1:]`.
- Return the node.

Below is an example of the algorithm's implementation in Python. I'm assuming there exists a `Node` class that can be used to create a binary tree.

```python
def construct_maximum_binary_tree(nums):
    if not nums:
        return None

    i = nums.index(max(nums))
    node = Node(nums[i])

    node.left = construct_maximum_binary_tree(nums[:i])
    node.right = construct_maximum_binary_tree(nums[i + 1:])

    return node
```

# Analysis

In each recursive step, we are doing three things -- searching for the maximum number, recursing on the left subarray, and recursing on the right subarray. The recurrence for this algorithm can be set up as $T(n) = 2T(\frac{n}{2}) + O(n)$ since we make two recursive calls and searching requires $O(n)$ time.

Using the [Master theorem](https://en.wikipedia.org/wiki/Master_theorem), this gives us a time complexity of $O(n \log n)$. Because we make around $\log n$ recursive calls, the space complexity is $O(\log n)$.

However, we have not considered the worst-case scenario. Imagine if we had an array that was sorted in ascending or descending order, such as `[1, 2, 3, 4, 5, 6]` or `[6, 5, 4, 3, 2, 1]`. Given an array of $n$ elements, we would recurse with $n - 1$ elements since either the left or right subarray would be empty. The recurrence would then turn into $T(n) = T(n - 1) + O(n)$. Because we have to make $n$ recursive calls, the time complexity would be $O(n^2)$ and the space complexity would be $O(n)$.
