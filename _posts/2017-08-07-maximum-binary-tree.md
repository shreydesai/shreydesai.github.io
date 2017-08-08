---
layout: post
title: "Maximum Binary Tree"
date: 2017-08-07
---

Recently, I started participating in the Leetcode weekly contests. So far, it has been a great experience - solving data structures and algorithm questions are hard enough, but under a time constraint, they can be even more challenging. However, I have noticed that the more problems I do, the more familiar other problems get. In the last contest, the second problem to solve was the <a href="https://leetcode.com/contest/leetcode-weekly-contest-44/problems/maximum-binary-tree/">Maximum Binary Tree</a>. It was pretty fun to solve, so I decided to write a small editorial on it!

# Problem

Given an integer array with no duplicates, a maximum binary tree building on this array is defined as follows:

- The root is the maximum number in the array
- The left subtree is the maximum number constructed from the left subarray divided by the maximum number
- The right subtree is the maximum number constructed from the right subarray divided by the maximum number

# Solution

Tree problems are usually recursive in nature, so the algorithm can be applied on each subtree. Therefore, the key to solving the problem is solving it for one case (correctly), so that it can be recursively applied to further cases.

So, this problem requires a couple of things. First, we need to find the maximum element in the array. This element will form the root of our tree. Second, we can partition the array into its left and right parts. The maximum element in the left and right parts will form the left and right subtrees, respectively. What's tricky is that an efficient solution will require us to find the maximum element and build the tree at the same time.

Because we form the root of the tree and then recurse on its left and right subtrees, that should remind you of a standard level order traversal. Essentially, we will be building the tree dynamically based on what the array contains in each recursive step.

Let's consider the base case. If we don't have any numbers in the array, then we can return `None`. This makes sense as if our initial array was completely empty, our tree would be empty as well. Now, we need to consider what our new array will look like in each recursive step. If the maximum element in an array is at index `k`, then the left partition will be the numbers from index `0` to `k`, exclusive, and the right partition will be the numbers from index `k + 1` to `n`, exclusive, where `n` is the number of elements in the array.

Therefore, we are left with the following algorithm:

- Check if the list is empty. If it is, then return `None`. This will presumably form the leaves of our tree.
- Find the index of the maximum number. Let's say it's at index `i`.
- Create a node whose internal element is `nums[i]`.
- Recurse on the left subtree with `nums[:i]` and recurse on the right subtree with `nums[i + 1:]`.
- Return the node.

```python
def construct_maximum_binary_tree(nums):
    if not nums:
        return None

    i = nums.index(max(nums))
    node = TreeNode(nums[i])

    node.left = construct_maximum_binary_tree(nums[:i])
    node.right = construct_maximum_binary_tree(nums[i + 1:])

    return node
```

You might be surprised that the algorithm is this short – I was too! It turns out binary tree problems are usually quite concise, but the algorithm itself requires some thought.

Now, let's consider the complexity of this algorithm. Finding the maximum element in each array is $O(n)$ because we do a linear search to find it. The array is not sorted, so it's impossible to do it faster, such as using binary search. In the average case, the height of our tree would be around $\log (n)$, but in the worst case, the height of our tree could be $n$. Because we perform the linear operation in each recursive call, the average case complexity would be $O(n \log (n))$ and worst case complexity would be $O(n^2)$.

I hope this article was helpful. When I solve more interesting problems in the weekly contests, I'll be sure to post about them as well!
