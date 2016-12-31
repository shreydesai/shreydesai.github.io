---
layout: post
title: "Fast Integer Sorting"
date: 2016-12-31
---

Sorting is a field in computer science that has much importance. Many complex problems can be solved in a much more efficient manner if their inputs are in sorted order. The time tradeoff between non-sorted and sorted lists is massive when considering problems like finding permutations, compressing data, or ordering datasets by a certain key.

Powerful algorithms like quicksort and merge sort are quick to take the spotlight as the fastest sorting algorithms out there. Even though both boast $O(n \log (n))$ complexities, they fall victim to certain inputs, such as mere integers.

Integer sorting is a topic within sorting that is of much interest because it beats the base $\Omega(n \log (n))$ sorting complexity that most comparison-based sorting algorithms require. Algorithms such as counting sort, bucket sort, and radix sort rely on non-comparison models that enable them to sort integers in $O(n)$ time. This article will introduce counting sort and radix sort, explaining the algorithm itself and how it performs relative to merge sort.

# Non-comparative sorting

As alluded to above, quicksort and merge sort rely on comparative models that require a base sorting complexity of $\Omega(n \log(n))$. This means that under the comparison model, it is impossible to have fewer than $n \log (n)$ steps. This is because the comparison model relies on two values - `True` and `False` - which dictate where a value goes in the sorted list.

Both quicksort and merge sort rely on inequality and equality operators to determine if one object is "greater" or "less" than another. The number of comparisons for any input of size $n$ is at most $\log (n!)$, which comes out to be $\Theta (n \log (n))$. This implies that if an algorithm uses comparisons to sort, then it has to be $\Omega (n \log (n))$.

In order to sort in $O(n)$ time, algorithms need to rely on other methods of sorting. When I first thought of this, I thought it was crazy because I believed numbers had to be compared with other numbers in a list for the list to be sorted. It was hard to imagine sorting a list by picking out a number and not comparing it to other numbers. But, this is where distributive sorting proved me wrong.

Distributive sorting algorithms find the key of an element, place the key in some data structure, then iterate through the data structure which effectively sorts the input.

# Counting sort

The best example of an easy to understand distributive sorting algorithm is counting sort. Let's say we have a list $A$ of the numbers $[3, 7, 6, 4, 6, 3]$. Define another list $A'$ that has length $max(A)$ and stores an empty list. The steps are as follows:

- For each index $i$ in $A$, add $A[i]$ to the list at $A'[A[i]]$

- Iterate through $A'$, collecting the numbers in each list

After step 1, $A'$ will look like $[[],[],[],[3,3],[4],[],[6,6],[7]]$. Iterating through each list in $A'$ gives us the sequence 3, 3, 4, 6, 6, 7, which is a sorted list!

Counting sort does not compare numbers with each other but rather places each number in an effective manner so iterating through the entire list yields a sorted form. Though a linear algorithm, it's quite inefficient in space. Imagine if we were sorting numbers in the 1000's - this would a ton of require auxiliary space. The solution to this is radix sort.

# Radix sort

Radix sort is another distributive sorting algorithm that sorts based on the base of a number, as the name suggests. Interestingly, it uses counting sort as a sub-function within the larger algorithm.

Radix sort exploits a fundamental property in numbers by sorting each digit at a time. By sorting the lowest significant digit to the highest significant digit in each step, the overall list will be sorted. Take the list $L$ defined as $[110, 493, 902]$, for instance.

- Sort $L$ by the one's place - $[110, 902, 493]$

- Sort $L$ by the ten's place (maintaining the one's sort) - $[902, 110, 493]$

- Sort $L$ by the hundred's place (maintaining the ten's sort) - $[110, 493, 902]$

Sorting each digit at a time results in the entire list being sorted. The function that sorts each individual digit is a modified version of counting sort, while radix sort manages the traversal of each digit. The pseudocode of the radix sort algorithm would look something like this:

```
for digit d in (least significant digit -> most significant digit):
  sort digits with respect to d
```

The complexity analysis for radix sort is a little complicated, but it comes out to be about $O(nk)$ where $k$ depends on the maximum number of digits a number has in the list to be sorted.

# Experiment

In order to satisfy my desire to compare the performance of merge sort and radix sort, I designed a small experiment to see how each would handle sorting a big list of integers. Each list had varying amounts of integers (from 10 to 1 million) with numbers up to five digits in size. Radix sort performed much better than merge sort, proving its finesse in sorting straight integers.

<img src="http://i.imgur.com/lVcP14A.png" width="95%">
