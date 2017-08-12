---
layout: post
title: "Microsoft Explorer Interviews"
date: 2016-10-28
---

Last week, I had the privilege of spending three days with Microsoft interviewing as a candidate for their Explorer program. The [Microsoft Explorer](https://careers.microsoft.com/students/explore) program is a 12-week paid summer internship where interns can _explore_ software engineering and program management. Groups of two to three interns are assigned a team and an impactful summer project.

After I received an email from my recruiter saying I was moving on to the final round interviews, I started finding material to prep with. The interviewing process was a little vague because it wasn't supposed to be 100% technical nor 100% behavioral. The intent of this article is to aid future Explorer candidates in better understanding what the on-site interviews are like and, most importantly, how to prepare for them.

## Pre-Interviews

I had about two weeks before I had to fly to Redmond, WA for the on-site interviews. During this time, I practically read every article and post on Quora and Glassdoor pertaining to the Explorer internship. After a lot of Internet browsing, I ended up compiling a large document with technical, product design, and behavioral interview questions.

Recruiters mentioned that Microsoft asked questions that tried to decipher the thought process of the candidate. Some questions would be intentionally ambiguous -- what was important was the approach, not the solution. These sorts of questions were difficult to prepare for because one couldn't change their thought process overnight. Nevertheless, I tried my best to prep -- each day, for about an hour, I went through interview questions in hopes that some would be asked in my on-site interviews.


## Interview 1

My first interviewer was a software engineer on Azure. He began by going down my resume, asking me probing questions about my projects and previous summer internship. Because I had worked on creating an internal RESTful API at [Sizzle](https://angel.co/sizzle), he asked me about the functionality surrounding HTTP methods like GET, POST, and PUT and the differences between POST and PUT.

In one personal project, I had used MongoDB and Redis, two popular databases. He asked me about the differences between the two, cases where someone would use one over the other, and the tradeoffs between SQL and NoSQL.

Continuing with this thread, he asked me an intriguing Redis question. If a cache write to Redis failed but a user requested said data, the back-end would look in the cache first. However, the key would not be there and the user would ultimately receive stale data. So, how would Redis handle these cases? At the time, I thought it might have something to do with timestamps, which was along the [right track](https://redis.io/commands/ttl).

We eventually got to the main interview question, which was to implement [Cron](https://en.wikipedia.org/wiki/Cron), the UNIX time-based job scheduler. I started with a mediocre solution but got closer to the optimal solution after discussing the problem with the interviewer.

I was pleasantly surprised by the caliber of these questions. It was _nothing_ like I had seen on Glassdoor -- I thought I was being interviewed for a regular software engineering intern position.

## Interview 2

For the second interview, I spoke with a software engineer on the Azure Automation team. We briefly went over my resume and my interviewer asked me some questions regarding the web crawling process used in [UT Courses](https://github.com/utexasdev/courses). On the whiteboard, he wanted me to explain at a high-level the algorithm of how my web crawler would traverse target websites.

Next, he asked me an interesting array question. We are given an array `nums` of unsorted integers. Given some arbitrary index `i`, if `nums[i] > 0`, then you can move forward `nums[i]` spaces. If `nums[i] < 0`, then you can move backwards `nums[i]` spaces. Determine if there is a loop between two arbitrary indices `i` and `j` in `nums`. If you're interested, this is the [Circular Array Loop](https://leetcode.com/problems/circular-array-loop/description/) question on Leetcode.

## Interview 3

The third interview went the best. The interviewer was a software engineering manager on the Azure Log Analytics/Security team, so he was doing a lot of fascinating work which he told me about when the interview was over.

The first question he asked me was interesting because it required me to solve a real-world problem using data structures. He asked me to implement an API that developers could use when tracking browser history. There were many different data structures one could use to solve this problem -- such as stacks or linked lists -- so this left room to discuss the pros and cons of each one. I had to define an interface, come up with tests, discover edge cases, and optimize the solution.

The second question was a basic string parsing problem. Given a string, print out all patterns of A's and B's. For example, the string `AaheaabelloaaabB` contains the patterns `Aa`, `aa`, `b`, `aaa`, and `bB`.

## Post-Interviews

Because each interview was 100% technical and I wasn't given a single behavioral or product design question, I was upset with how the interviews went. Every single question I was asked was something I had never encountered in my preparation.

In retrospect, this was an important lesson to learn because interviewers will try to ask questions that is outside the scope of what a candidate has prepared for. It is a _very_ useful skill to fully understand a problem, come up with various solutions, and optimize them when requirements change.

After the interviews, we were given a free Microsoft jacket and $55 to explore Seattle! I had a couple of friends studying at the University of Washington, so I went to visit them. We enjoyed a meal at [Little Thai](https://www.yelp.com/biz/little-thai-restaurant-seattle) on Microsoft's expenses.

Ultimately, the experience was phenomenal. It was my first time being flown out to an on-site interview, staying at an extravagant hotel with all expenses paid, and speaking with candidates all over the nation.

## Decision Day

It took exactly a week for a recruiter to get back to me with a decision. I was getting very nervous because other candidates had already heard back with their decisions by the time I received mine.

Fortunately, I received an offer! I was ecstatic and shouted a couple of times in my room before settling down. Based on my preferences and interview performance, I was placed on the [Azure Stack](https://azure.microsoft.com/en-us/overview/azure-stack/) team in the Cloud Enterprise and Management group. This was exactly where I wanted to work. I had worked with AWS at my previous internship, so I wanted to get experience with Azure.

## Concluding Remarks

I hope this article shed some light on what the Microsoft interview process is like and what to expect. Generally speaking, interviewers ask questions that are tailored to each candidates' experiences and resume. It should not have come as a surprise that I didn't get standard string or array reversal questions. Don't be discouraged if some of my questions seemed difficult -- there's a high chance yours might be different.

Here's the takeaway -- Microsoft doesn't look for candidates that memorize and regurgitate knowledge, but rather those that show a strong potential to grow and learn. The best interview preparation you can do is tackling technical, behavioral, and product design questions you have _never_ seen before. Catching someone off-guard offers a unique way into probing his or her thought process and analytical skills.

If you have questions about submitting an application, the interview process, or the Explorer program in general, don't hesitate to reach to me on social media!
