---
layout: post
title: "Introduction to Tree Maps"
date: 2016-09-29
---

With CS 314 (Data Structures) in full force, I came across a data structure that I had not heard about before - Tree maps. I decided to write an article on it in hopes that it would bolster my knowledge about the subject and potentially help someone else that was wondering about it. I'll discuss the differences between tree maps and hash maps, why using a tree map would be useful in comparison to its siblings, and finally there will be demonstrations of how to do basic operations with tree maps.

## Introduction

In my previous projects, I had worked a lot with hash maps before - these are key-value stores where each object is assigned a key in some storage container and the object's data is the corresponding value. Tree maps are similar - but also very different! When keys are assigned in hash maps, the keys themselves are stored in a random order. This implies that access to the keys will not be that expensive in terms of Big O because there isn't a specific order the search function must use in order to retrieve a key.

While hash maps use arrays as their internal storage container, tree maps in Java use Red-Black trees. The difference in storage containers, therefore, is the primary rift between these two data structures.

When objects are stored inside hash maps, the algorithm that stores the object inside the container must generate some sort of hash key, which is a unique identifier that will tell the internal container where the object will be stored. Because each object has a unique hash key (hopefully), the amortized time complexity for retrieving a key is $O(1)$. However, as I noted above, these keys are not necessarily in sorted order.

Tree maps, on the other hand, ensure that the keys are stored in sorted order, so instead of generating hash keys for the objets, they instead rely on the content that is being stored. For instance, if an `Integer` is being stored inside the tree map, then the algorithm will call the `compareTo` method to determine where the `Integer` should be stored. Just like binary search trees, Red-Black trees store their nodes in accordance to a rigid policy, but they are balanced binary trees. Finally, the time complexity for basic search operations would be $O(log(n))$, which is a little worse than hash maps, but still pretty good.

## Use cases

If the main difference between tree maps and hash maps is the fact that the keys are stored in tree maps, then this would be the primary reason to use tree maps. In most cases, however, developers might not need their keys to be sorted, so a hash map would suffice. Yet, there are probably instances in which sorted keys would be useful.

Consider a program that emulates a dictionary - there are basic alphabetical letters (A, B, C, ...) and these can be keys. The values that are associated with these keys are words themselves, particularly, those that start with the letter of the key. This would be a great instance to use tree maps because alphabetical order has a specific ordering; this ordering can only be maintained if the keys are sorted, which a tree map does. Though it looks very strange, it is possible to have a tree map with `String`s as keys and `ArrayList<String>`s as values. In the past, I've mostly used a `String` and `Integer` pair, but there really are infinite possibilities.

## Tree Map Demonstrations

### `put` operation

Inserting a value into the tree map is quite simple - this is where you use the `put` operation. `put` takes in two parameters - the key that needs to be stored and the value that will be mapped at the key.

```java
Map<String, ArrayList<String>> map = new TreeMap<String, ArrayList<String>>();

ArrayList<String> words = new ArrayList<String>();
words.add("snake");
words.add("serpent");

map.put("S", words);
```

If you `put` a key-value pair that has already been stored inside the tree map, then this will generate a "collision," which basically means you're trying to store a value with the key that is equal to an existing key in the data structure. Instead of appending the object to a linked list or using linear probing techniques, the tree map will just replace the old value with the new value.

### `remove` operation

Removing a key from the tree map will remove both the key and the value associated with it. One important caveat I found out recently was that if you try to remove a key that is not stored in the tree map, then this will just return `null`. I thought it would throw an exception, but I guess not.

```java
map.remove("S");
map.remove("Z"); // returns null
```

### Iterating

There are a couple of ways to iterate through a tree map. I'll discuss two approaches - using the built-in `iterator` and using a `for each` loop.

With the `iterator`, there are a couple of ways you can iterate through the tree map. It really depends on what you're trying to do, though.

```java
Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
  String value = iterator.next();
}

// or

Iterator<Map.Entry<String, ArrayList<String>>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<String, ArrayList>> entry = iterator.next();
}
```

A for each loop can also be used here. The iterator might be better, though, because `keySet()` or `entrySet()` will be called every time the loop iterates through an element, which is a bunch of method calls that do not need to happen. I'm pretty sure these are just $O(1)$, though, but still.

```java
for (String s : map.keySet()) {
  // body of loop
}

// or

for (Map.Entry<String, ArrayList<String>> entry : map.entrySet()) {
  String s = entry.getKey();
  ArrayList<String> list = entry.getValue();
}
```
