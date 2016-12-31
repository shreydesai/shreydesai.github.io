---
layout: post
title: "Top K Frequent Elements"
date: 2016-12-03
---

In the beginning of the semester, I was truly perplexed as to why I had to take a course on data structures. The first thing our professor told us was that data structures were, by definition, variables that held other variables. Yet, there seemed to be a craze about the importance of data structures and algorithms, especially in software engineering.

Back when I took AP Computer Science, I found most of my computer science problems being solved by the most minimal data structure out there - the array. Because an array could have many declared types, it could hold all sorts of data. Arrays were easy to create, didn't have that many searching procedures, and were - most importantly - the only data structure that I was aware of.

I had heard of linked lists, trees, and hash tables, but it wasn't really clear how any of these data structures could provide a better interface than the standard array. Now that the semester is almost over, I realize that there is more to life than the array. This epiphany comes from one of the first interview problems I ever attempted - finding the top $k$ frequent elements in a list of numbers.

# Naive Algorithm

Initially, the top $k$ frequent elements problem seems quite straightforward. Given a list of numbers, say $[1, 1, 1, 2, 2, 3]$, find the top $k$ frequent element. For instance, the top $2$ frequent element is $2$, while the top $3$ frequent element would be $3$.

When I approached this problem last summer, I had a general idea of how I was going to solve it. First, I would count up the frequencies of each number, ensure I could map each number to its frequency, and then find the top $k$ frequency based on this analysis. Though it seemed obvious in my head, the implementation turned out to be quite difficult. Further, Leetcode wouldn't even accept my answer in the beginning because its time complexity was quite large, which I will discuss below as well.

Initially, I solved this problem in Python. I used two data structures - the list and dictionary. The list is pretty much like a standard, re-sizable array, similar to the `ArrayList` in Java. The dictionary is essentially an hash table that stores its keys in an unsorted manner.

The first part of the algorithm was to create a dictionary that mapped each number to its respective frequency.

```python
freqs = {}
for num in nums:
  if num not in freqs.keys():
    freqs[num] = 1
  else:
    freqs[num] += 1
```

Because the dictionary is not bi-directional, it's not possible to get the frequency associated with a particular value. The key-value pairs therefore need to be inverted by creating another dictionary that does this. Because some numbers might have the same frequency, the value in the key-value pair is a list that consists of all numbers sharing a particular frequency.

```python
inverted_freqs = {}
for key in freqs:
  if freqs[key] not in inverted_freqs.keys():
    inverted_freqs[freqs[key]] = [key]
  else:
    inverted_freqs[freqs[key]].append(key)
```

Finally, because the dictionary does not store its keys in sorted order, the keys must be sorted in order to get the top $k$ frequent element. If the keys were sorted in reverse order, or from largest to smallest, then the algorithm could find the top $k$ frequent element by querying the `inverted_freqs` map.

```python
sorted_freqs = sorted(inverted_freqs.keys(), reverse=True)
kth_frequency = sorted_freqs[k - 1]
kth_element = inverted_freqs[kth_frequency]
```

Put together, the algorithm looks like this:

```python
def top_k_frequent(nums, k):
    freqs = {}
    for num in nums:
        if num not in freqs.keys():
            freqs[num] = 1
        else:
            freqs[num] += 1

    inverted_freqs = {}
    for key in freqs:
        if freqs[key] not in inverted_freqs.keys():
            inverted_freqs[freqs[key]] = [key]
        else:
            inverted_freqs[freqs[key]].append(key)

    sorted_freqs = sorted(inverted_freqs.keys(), reverse=True)
    kth_frequency = sorted_freqs[k - 1]
    kth_element = inverted_freqs[kth_frequency]

    return kth_element
```

Though this solution works, it's not that efficient.

- Constructing the frequency map is approximately $O(n)$ as iterating through a list of $n$ elements takes roughly $n$ steps.

- Inverting the frequency map is also $O(n)$ where $n$ is the number of keys in the map.

- Most of the algorithm's overhead would probably come through sorting, which is $O(nlog(n))$ assuming the frequencies are not in natural order.

- Querying the frequency map does not depend on the size of any list, so it would be a $O(1)$ operation.

Because there are two $O(n)$ operations, this could be consolidated into $O(n)$ as only the upper bound function matters. Also, the $O(1)$ operation is irrelevant in the grand scheme of the algorithm because there are other operations that depend on $n$. The only consideration now is the comparison of the $O(n)$ and $O(nlog(n))$ operations.

If $f_1(x)$ is $O(g_1(x))$ and $f_2(x)$ os $O(g_2(x))$, then $(f_1+f_2)(x)$ is $O(\max(g_1(x), g_2(x)))$. Because $n \le nlog(n)$, the overall time complexity of this algorithm would approximately be $O(nlog(n))$.

# Heaps to the Rescue

While $O(nlog(n))$ seems pretty good, there is a better approach. Before I learned about heaps, I would have never come up with this solution. This problem is a great example of something that had a better solution with a different data structure, which is why it's so important to give thought into using the best data structure for a particular problem.

Heaps are complete binary trees that maintain either the max or min property. For max heaps, the root node of any sub tree is greater than or equal to each of its child nodes. By analogy, for min heaps, the root node of any sub tree is less than or equal to each of its child nodes. This property makes it very trivial to continually query for the minimum value in some batch of data, which is exactly what this problem calls for.

The algorithm using heaps would be quite simple. We would add element and its frequency to a map, insert the elements to a max heap, and then remove $k$ elements from the heap. The $k$ elements removed from the heap would be the maximum elements in the original list, so the algorithm checks out. Note: The only reason I implemented this in Java was because Python doesn't have nice built-in data structures like Java.

First, we should initialize a list to store the top $k$ frequent elements. Next, the element-frequency pairs can be built by creating a hash map. I used a Linked List for the initial list because each add is guaranteed to be $O(1)$, but `ArrayList` adds might be $O(n)$ because of resizing.

```java
List<Integer> topK = new LinkedList<Integer>();
Map<Integer, Integer> freqs = new HashMap<Integer, Integer>();
for (int i = 0; i < nums.length; i++) {
  if (freqs.get(nums[i]) == null) {
    freqs.put(nums[i], 1);
  } else {
    freqs.put(nums[i], freqs.get(nums[i] + 1));
  }
}
```

The `PriorityQueue` class in Java uses a heap as its underlying storage container. However, the way that we add elements to the heap is a little nuanced. We need to compare which frequency of two numbers is greater, which is why a unique `Comparator` must be defined in the constructor.


```java
Queue<Integer> heap = new PriorityQueue<Integer>(nums.length,
  new Comparator<Integer>() {
    @Override
    public int compare(Integer i1, Integer i2) {
      return freqs.get(i2) - freqs.get(i1);
    }
});
```

Finally, the elements can be added to the heap, removed $k$ times, and added to the `topK` list, which now contains the top $k$ frequent elements.

```java
Iterator<Integer> it = freqs.keySet().iterator();
while (it.hasNext()) {
  Integer elem = it.next();
  heap.add(elem);
}

for (int i = 0; i < k; i++) {
  Integer elem = heap.remove();
  topK.add(elem);
}
```

Put together, the algorithm looks like this:

```java
public List<Integer> topKFrequent(int[] nums, int k) {
  List<Integer> topK = new LinkedList<Integer>();

  Map<Integer, Integer> freqs = new HashMap<Integer, Integer>();
  for (int i = 0; i < nums.length; i++) {
    if (freqs.get(nums[i]) == null) {
      freqs.put(nums[i], 1);
    } else {
      freqs.put(nums[i], freqs.get(nums[i] + 1));
    }
  }

  Queue<Integer> heap = new PriorityQueue<Integer>(nums.length,
    new Comparator<Integer>() {
      @Override
      public int compare(Integer i1, Integer i2) {
        return freqs.get(i2) - freqs.get(i1);
      }
  });

  Iterator<Integer> it = freqs.keySet().iterator();
  while (it.hasNext()) {
    Integer elem = it.next();
    heap.add(elem);
  }

  for (int i = 0; i < k; i++) {
    Integer elem = heap.remove();
    topK.add(elem);
  }

  return topK;
}
```

Though the algorithm seems quite long, it's actually much more efficient than the previous one. The static typing nature of Java might trick you into believing you're doing more work than you're supposed to, but it's really not the case! Let's take a look at the individual steps.

- Constructing a map of frequencies, as demonstrated in the previous algorithm, is $O(n)$. This cannot be avoided or minimized as we must read each element at least once in the algorithm.

- Declaring the heap, including creating the comparator, would be $O(1)$. However, adding elements to the heap would be $O(nlog(k))$ where $n$ is the number of unique elements in the list and $k$ is the number of elements already present in the heap.

- Dequeuing the elements from the heap would be $O(n)$ where $n$ is $k$.

The total complexity of this algorithm would be about $O(n + nlog(k))$ or $O(nlog(k))$ for simplicity, which definitely beats the algorithm above. The algorithm was also quite trivial when a heap was used because we were directly exploiting the property of a max heap.

In conclusion, the intent of this article was to demonstrate that algorithmic thinking should not be a top down approach, where we use the same data structure or approach for each problem. Rather, data structures exist because we can organize and retrieve data in different ways, some vastly efficient than others. In this case, the heap was far better than an array or hash map alone.
