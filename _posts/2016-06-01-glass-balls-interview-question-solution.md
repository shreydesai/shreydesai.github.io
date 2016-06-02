---
layout: post
title: "Glass Balls Interview Question Solution"
date: 2016-06-01 01:51:00
categories: Algorithms
featured_image: /images/cover.jpg
---

As I was waiting for a Skype call with [Xperii](http://xperii.com/) - my first technical interview ever - I was nervous to say the least. Upon reading many Glassdoor interview posts, it seemed that startups gave harder questions to their candidates in comparison to bigger companies such as Amazon, Google, Facebook, and Microsoft. Lo and behold, the first question I received was the infamous glass balls question, which was a daunting and challenging problem. After a couple of weeks of introspection, I have come up with a couple of solutions.

## The Problem

Say you are on a 100-story building and are equipped with two glass balls. These balls will break at some floor $F$ and every single floor greater than or equal to $F$. However, the balls are reusable and will not break at any floor less than $F$. Therefore, the task is to determine which floor is the "breaking point."

## Solution 1: The Naïve Solution

My first instinct was to recount the binary search algorithm, which runs in $O(log(n))$ time and can find an element - or a floor in this case - quickly. The problem with this was that I missed the detail that I only had two balls, and if the binary search could not find the given floor in two tries, I would run out of balls. This solution was not equitable as a result.

The next attempt would be a variation of the linear search, which is much slower. We could drop the ball at each floor and if the ball breaks at some floor $F$, then $F$ would be the designated "breaking point" of the ball.

Even though this solution worked, it was very ineffective. For starters, its time complexity was $O(n)$, which means at at most it would take $n$ tries to find the floor at which the ball breaks. It seems intuitive that the algorithm should find a lower breaking point faster than a higher breaking point, which this solution did not account for.

## Solution 2: The Interval Solution

This is where I got stumped and was unable to finish the problem. Yet after weeks of pondering, I am now comfortable to share the rest of the solutions to this problem. In the last solution, the steps were equal to $1$, i.e. we tested floor $1$ and then $2$ and then $3$. What if the steps could be larger? 

If we dropped the ball at floor $50$, there would be two possibilities. If the ball broke, then we would know 

$$1 \leq F \leq 50$$

If the ball didn't break, then we would know 

$$51 \leq F \leq 100$$

Afterwards, we would use the naïve solution with a step of $1$ to find where the ball breaks since any other step might miss where the breaking point actually is. Even though the complexity of this algorithm is $O(\frac{n}{2})$, it is still ineffective because the initial step of $50$ was quite large - the question then remains how much we should step by.

This requires writing a function that resembles the maximum steps that we have to take and then finding the minimum of this function. The minimum would represent how much we should step by originally.

Given a $N$-story building and $S$ steps, the maximum number of steps required with the algorithm described above would be represented by the following:

$$F(S) = \frac{N}{S} + S + 1$$

For example, if the breaking point of a $100$-story building was $100$, then we would step by $10$ until the ball breaks at floor $100$ - the test sequence would be floor $10$, $20$, $30$, and so on. Then, because the ball breaks at $100$, we would go from $91$ to $100$ with the second ball to find the precise break point. Note that we would not cover floor $90$ because this was already tested with the initial step of $10$. 

Because we need to find the minimum number of steps to achieve the best value with respect to the naïve steps that we have to take, the minimum of the function needs to be found. This can be found easily through calculus.

$$\frac{dF}{dS} = 1 - \frac{N}{S^{2}}$$
$$\frac{N}{S^{2}} = 1$$
$$S^{2} = N$$
$$S = \sqrt{N}$$

Therefore, the minimum number of intervals required for a $100$-story building would be $\sqrt{100}$, or $10$. Using a skip interval of $10$, we can find a breaking point with the two glass balls. Unlike the naïve solution found above, this algorithm would be much faster.

## Conclusion

This question was tough and it was even harder because I was not expecting to be given such a mathematical question for a software engineering intern position. Because theoretical computer science is intertwined with mathematics, it might beehove undergraduates to study up on algorithms and even brush up on important topics in algebra, trigonometry, and calculus.