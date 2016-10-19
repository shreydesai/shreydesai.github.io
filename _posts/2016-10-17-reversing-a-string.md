---
layout: post
title: "Reversing a String"
date: 2016-10-17
---

Reversing a string is one of the most common questions asked for software engineering interviews. As I thought about the problem more and more, I realized that there were much more solutions than I had initially considered. During a technical interview, a candidate should continue to optimize his or her solution more and more, to the point where the candidate reaches the optimal solution or at least gets close enough. Because of the large amount of string reversal algorithms out there, it's very doable to impress an interviewer. This article will discuss three approaches to reversing a string and its time complexities.

## Solution 1: Reverse Iteration

The best way to code an algorithm is to think about it in plain terms. It's very easy to get caught up with different data structures and standard library functions. Drawing out the steps of an algorithm with pen and paper often yields a lesser amount of errors than actually coding.

However, sometimes this might not produce the best algorithm. If you think about reversing a string intuitively, you might consider starting at the end of the string and working backwards until you reach the beginning. I guarantee that is how most people think about it in their heads. This is a great solution for our brains, which work fast, but the computer might not be able to process this algorithm given a very large string. Nonetheless, reverse iteration is still a solution.

```java
private String reverse(String s) {
  String reversed = "";
  for (int i = 0; i < s.length(); i++) {
    char last = s.charAt(s.length() - i - 1);
    reversed += "" + last;
  }
  return reversed;
}
```

This algorithm is quite simple. It creates a temporary `String` variable, which ends up storing all of the characters of the original `String` in backwards order. However, this solution is definitely $O(n^2)$, which means each time the string length doubles, the time to process it will quadruple. Many people are quick to say the algorithm is $O(n)$, but they often miss that string concatenation itself is an $O(n)$ operation.

Each time a string is concatenated to another string, the new string is a combination of the previous string plus the new string. This subtle detail is often masked by the `+=` operator. However, consider what is going on with this algorithm given the `String` "hello":

```
'o'
  'o' + 'l'
    'ol' + 'l'
      'oll' + 'e'
        'olle' + 'h'
          'olleh'
```

String concatenation is an $O(n)$ operation because the different parts of the string must be added together $n$ times where $n$ is the size of the string. This occurs because strings in Java are immutable, so a new `String` must be created every single time. Using the `StringBuilder` class can easily resolve this problem.

```java
private String reverse(String s) {
  StringBuilder sb = new StringBuilder();
  for (int i = 0; i < s.length(); i++) {
    char last = s.charAt(s.length() - i - 1);
    sb.append(last);
  }
  return sb.toString();
}
```

The `append` method is $O(1)$ because all the `StringBuilder` does is add the new character to the end of a mutable sequence of characters, namely a `char` array. This appending happens $n$ times and the `toString` method is also a $O(n)$ operation, so the overall time complexity using the `StringBuilder` would probably be $O(n)$, more or less.

## Solution 2: Recursion

Recently, I discovered a pretty neat recursive solution to reversing a `String`. In CS 314, one of our recursion assignments involving implementing this solution, so I would deem this article a good place to share it!

Understanding recursion itself can be quite difficult because it requires the use of an implicit stack. The best way to approach recursive problems is to draw out the stack, understand the base case, and pray that the solution works.

```
f(hello)
  o + f(hell)
    ol + f(hel)
      oll + f(he)
        olle + f(h)
          olleh
```

When reversing a string, the general, recursive idea is to extract the last character, call the function on the rest of the string, and stop when the length of the original string is one. In Java, it would look like this:

```java
private String reverse(String s) {
  String rev = "";
  if (s.length() == 1) {
    rev += s;
  } else {
    int i = s.length() - 1;
    String rest = s.substring(0, i);
    rev = s.charAt(i) + reverse(rest);
  }
  return rev;
}
```

Unfortunately, the recursive approach does not really improve the time complexity because the `substring` method itself is $O(n)$ as it has to process through at most $n$ characters in a `String`. Also, there are at most $n$ recursive calls to the function, so this is a classic $O(n^2)$ algorithm. However, it's pretty cool to utilize the power of recursion to solve a simple problem like this. The interviewer might also be impressed that you have some tricks up your sleeve.

## Solution 3: In-place Modification

The in-place modification solution takes advantage of the fact that a `String` is merely a bunch of `char` elements combined together. Therefore, the main intent with this solution is to keep swapping the `char`s in the front and end until it reaches the middle.

```java
private String reverse(String s) {
  StringBuilder sb = new StringBuilder();
  char[] elems = s.toCharArray();

  int mid = s.length() / 2;
  int size = elems.length - 1;

  for (int i = 0; i < mid; i++) {
    char temp = elems[i];
    elems[i] = elems[size - i];
    elems[size - i] = temp;
  }

  for (int i = 0; i < elems.length; i++) {
    sb.append(elems[i]);
  }

  return sb.toString();
}
```

This algorithm makes a couple of passes through the sequence of characters, but it's still $O(n)$ because only the dominant term, namely $n$, is considered when doing time complexity analysis. However, if memory is a problem, this might not be the best solution. In this algorithm, two arrays are created: one to reverse the elements and one to build the string. If the `char` class could magically turn their arrays into `String`s, this might be a better solution.

## Bonus Solution: Python Indexing

When solving technical questions, there are many standard library functions that often complete the solution for you. For instance, I don't think JavaScript has a string reverse function, but it has a function to reverse an array. Someone could theoretically create an array of characters, reverse it, build the `String`, and return it. While this is interesting, interviewers want to go beyond the "wall of abstraction," so this will usually be considered invalid.

However, there is one library function that is insane. Python has some unique string and list indexing techniques. Many languages, including Java, allow developers to view elements at a particular index in an array (given that it is within the bounds). Python's string and list indexing goes a step further by allowing developers to put weird indices in and get cool answers.

Consider the following examples:

```python
>>> nums = [1, 2, 3, 4, 5]
>>> nums[2]
3
>>> nums[-1]
5
>>> nums[-2]
4
>>> nums[::-1]
[5, 4, 3, 2, 1]
```

The `[::-1]` indexing operation is pretty cool. Python allows you to put in a third parameter when doing indexing, which is a "skip" parameter. For instance, `[0:5:2]` will iterate through indices 0 to 5 by skipping 2 each time. For the variable `nums`, this would return `[1, 3, 5]`.

Because `String`s are iterables in Python, the same logic can be applied here. I'm not really sure how this works, but I believe `[::-1]` iterates through each character by reverse skipping each one. Therefore, using that indexing operation on `String`s is simply magical:

```python
>>> s1 = 'hello world'
>>> s2 = 'github is cool'
>>> s3 = 'i love python'
>>> s1[::-1]
'dlrow olleh'
>>> s2[::-1]
'looc si buhtig'
>>> s3[::-1]
'nohtyp evol i'
```

I looked up the time complexity for this operation. The Python documentation said that all indexing methods on `String`s are $O(n)$, which makes sense because each character in a `String` must be traversed at least once to construct a reversed `String`. However, I'm not sure if there is additional overhead through operations like string concatenation.
