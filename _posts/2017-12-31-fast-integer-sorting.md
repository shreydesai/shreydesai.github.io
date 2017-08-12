---
layout: post
title: "Fast Integer Sorting"
date: 2016-12-31
---

Sorting has remained an important topic of discussion in computer science. Sorting is extremely useful in canonicalizing data and producing human-readable output. Further, recursive and dynamic programming algorithms have considerable speed improvements when given sorted inputs.

Powerful sorting algorithms like [Quicksort](https://en.wikipedia.org/wiki/Quicksort) and [Merge sort](https://en.wikipedia.org/wiki/Merge_sort) are widely used in the industry. Echoing the words of many interviewers -- can we do better?

Surprisingly, yes, _kind of_. If we are allowed to make certain assumptions about our data, then we can exploit these characteristics using an asymptotically superior sorting algorithm. Algorithms to sort integers have beat $O(n \log n)$ algorithms, completing the task in linear time. This article will discuss two such algorithms -- [Counting sort](https://en.wikipedia.org/wiki/Counting_sort) and [Radix sort](https://en.wikipedia.org/wiki/Radix_sort) -- and how they stack up against Quicksort and Merge sort.

## Non-comparative Sorting

Quicksort and Merge sort are comparative sorting algorithms, which means they use a comparative model to sort data. The comparisons made are generally less than, equal to, or greater than -- these are the three operators that define whether an element `x` comes after an element `y`.

The number of comparisons for any input of size $n$ is at most $\log (n!)$. According to [Stirling's approximation](https://en.wikipedia.org/wiki/Stirling%27s_approximation), the lower bound is $\Omega (n \log n)$. This means that if an algorithm relies on the comparative model, it will require -- at the least -- $n \log n$ operations.

So, how do we do better? We have to move away from the comparative model. [Distribution sorts](https://en.wikipedia.org/wiki/Sorting_algorithm#Distribution_sort) have an answer to this question. Instead of directly comparing two elements, distribution sorts distribute their input to auxiliary data structures and collect them into a sorted output. Two popular distribution sorting algorithms include [Counting sort](https://en.wikipedia.org/wiki/Counting_sort) and [Radix sort](https://en.wikipedia.org/wiki/Radix_sort), both of which are discussed below.

## Counting sort

Let's say we had an input of `[3, 7, 6, 4, 6, 3]`. A human might sort this array by collecting like terms and concatenating the results together. For example, there are two `3`s, one `4`, two `6`s, and one `7`. If we keep track of how many times a number occurs, we can easily get a sorted output.

Counting sort does exactly this. From the unsorted input, the count of each number is kept track of in a separate array. The counts array is then iterated over to yield a sorted output. There are no comparisons required, breaking the requirement for a comparative model. This is how the algorithm could be implemented in Python:

```python
def counting_sort(nums):
    buckets = [0 for i in range(max(nums) + 1)]

    for i in range(len(nums)):
        buckets[nums[i]] += 1

    k = 0

    for i in range(len(buckets)):
        for j in range(buckets[i]):
            nums[k] = i
            k += 1
```

Iterating over the unsorted input of $n$ elements is $O(n)$. Afterwards, we iterate over the counts array, which is $O(k)$ where $k$ is the maximum element in the unsorted input. Therefore, the time complexity is $O(n + k)$ and the space complexity is $O(k)$.

While this algorithm shines when the maximum element is significantly smaller than the size of the input, it can degenerate quite quickly. For example, consider an input of `[3, 2, 1000000]`. This is quite trivial to sort, but Counting sort would require _at least_ $1000000$ operations to sort the input.

## Radix sort

Radix sort exploits a key characteristic of numbers in order to sort its input. The algorithm sorts each number digit-by-digit starting from the least significant digit to the most significant digit. To understand how this works, consider the input `[274, 61, 2, 4, 80, 93]`. I will pretend the input is padded with extra zeros so you can get a better understanding of how this works.

So, our unsorted input is `[274, 061, 002, 004, 080, 093]`. First, let's sort the input based on each number's one's digit -- `[080, 061, 002, 093, 004, 274]`. Next, the ten's digit -- `[002, 004, 061, 274, 080, 093]`. Finally, the hundred's digit -- `[002, 004, 061, 080, 093, 274]`. Without the extra padding, our output is `[2, 4, 61, 80, 93, 274]`, which is sorted.

Radix sort operates similarly to counting sort in that it groups numbers based on their digits. Interestingly enough, some implementations use counting sort as a subroutine to do so. In Python, Radix sort would look something like this:

```python
import math

def digits(x):
    return int(math.log10(x)) + 1

def counting_sort(nums, exp, base):
    sorted_nums = []
    counts = [[] for i in range(base)]
    for i in range(len(nums)):
        index = (nums[i] % exp) // (exp // base)
        counts[index].append(nums[i])
    for i in range(len(counts)):
        sorted_nums.extend(counts[i])
    return sorted_nums

def radix_sort(nums):
    sorted_nums, base = nums, digits(max(nums))
    for i in range(base):
        exp = base ** (i + 1)
        sorted_nums = counting_sort(sorted_nums, exp, base)
    return sorted_nums
```

Iterating over the unsorted input of $n$ elements is $O(n)$. Because we have to do this for $k$ digits, the time complexity is $O(kn)$ and space complexity is $O(k + n)$. In the average case, this is an extremely fast sorting algorithm. Even though Radix sort is specifically an integer sorting algorithm, many specially formatted floating numbers and strings have underlying integer representations, so the algorithm can still be applied.

## Sorting Experiments

Distribution sorts should be faster than comparison sorts when we make make certain assumptions about our input. For the purposes of the experiment, assume that we are given an array of unsorted integers where the integers range from `0` to `100000`.

<img src="{{ site.baseurl }}/images/2017-12-31-1.png">

The distribution algorithms outperformed the comparative algorithms when sorting large inputs. As expected, Quicksort performed better than Merge sort. Further, Radix sort was slightly faster than Quicksort as the input size grew. Counting sort was quite fast, but note that its speed came at the cost of considerable memory usage, which is also an important factor to consider.

The experiment code is below. Note: The algorithms themselves are not included in this snippet. Please refer to [my repository](https://github.com/shreydesai/algorithms/tree/master/sorting) to retrieve them.

```python
import time
import random

MAX_NUMBER = 100000
MAX_SIZE = 5500000
STEP_SIZE = 500000

def generate_array(size):
    return [random.randrange(0, MAX_NUMBER + 1) for i in range(size)]

def timer(f, *args, **kwargs):
    start = time.time()
    f(*args, **kwargs)
    return time.time() - start

for i in range(1, MAX_SIZE, STEP_SIZE):
    nums = generate_array(MAX_SIZE)

    alg1 = timer(counting_sort, nums)
    alg2 = timer(radix_sort, nums)
    alg3 = timer(merge_sort, nums)
    alg4 = timer(quicksort, nums)

    print('{},{:f},{:f},{:f},{:f}'.format(
        i, alg1, alg2, alg3, alg4
    ))
```
