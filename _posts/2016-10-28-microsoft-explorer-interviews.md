---
layout: post
title: "Microsoft Explorer Interviews"
date: 2016-10-28
---

Last week, I had the opportunity to spend three days with Microsoft interviewing as an intern for their Explorer program. For those that are not familiar with this program, it is a 12-week summer internship where interns can "explore" software engineering and program management by working on a particular product with a team at Microsoft. After I received an email from my recruiter saying that I was invited to the on-campus interviews, I frantically started finding material to prep for. The Explorer interviews are a little vague because they aren't 100% technical nor 100% behavioral - a unique blend of both. The intent of this article is to aid future Explorer candidates in better understanding what the on-site interviews are like and how to prepare for them.

## Pre Interviews

I had about two weeks before I had to fly to Redmond for the on-site interviews. During this time, I read every article on Quora regarding Explorer interviews, scoured Glassdoor for interview questions I could prepare for, and consulted with previous interns regarding their experience. I ended up compiling a large document with a bunch of interview questions - technical, product design, behavioral - I had it all.

The recruiter told us that Microsoft asked questions that got at the thought process of the candidate. So, some questions would obviously be ambiguous; what was important was not the solution but rather how a candidate approached the question. This sort of creativity was difficult to prepare for because one cannot obviously increase the amount of creativity he or she has during the span of a week. Nevertheless, I tried my best to prep myself - each day, for about an hour, I would go through interview questions in hopes that some would be asked in my on-campus interviews.

## Interview 1

My first interviewer was a software engineer on Azure. He started the interview by going down my resume, asking me probing questions about my projects and previous summer internship. Because I had worked on creating an internal RESTful API at Sizzle, he asked me about different HTTP requests, namely GET, POST, and PUT.

I had also worked with different databases - MongoDB and Redis - in my personal projects. He asked me about the differences between MongoDB and Redis, in which cases I would use one over the other, and why NoSQL might be a better option than SQL.

Afterwards, he asked me an interesting Redis question. Basically, if the caching process to Redis (from the main database) failed but the user requested data, the server would look for the key in Redis first. However, assuming the caching process failed, the key in Redis would not be updated and therefore the user would get old data.

We eventually got the main question for the interview, which was to implement Cron, a time scheduling manager on Linux systems. Cron is useful in scenarios where tasks have to be completed at a particular schedule; essentially, once a particular "time" is reached, its respective function is called.

I was pleasantly surprised by the caliber of these questions. It was nothing like I had seen on Glassdoor, but I was reasonably prepared to answer them. In retrospect, it makes sense that these questions were asked because they were tailored to my background, and more specifically, my resume.

## Interview 2

For the second interview, I spoke with a software engineer on the Azure Automation team. We briefly went over my resume; he was interested in learning more about a web crawler that I made as a part of one of my projects. On the whiteboard, he wanted me to explain the general algorithm of how my web crawler would traverse the inputted website.

Next, he asked me a question about arrays, which was a graph question in disguise. High level, it basically asked me to find whether a cycle existed in the graph. Formally, it said that given an index $i$ in array $A$, if $A[i] > 0$, then move forward $A[i]$ spaces. If $A[i] < 0$, then move backward $A[i]$ spaces. Determine if there is a loop where there exists a path between two indices $i$ and $j$ in $A$ where $i,j \in D$.

## Interview 3

The third interview went the best. The interviewer was a software engineer on the Azure Log Analytics/Security team, so he was doing a lot of cool work, which he told me about when the interview was over.

The question he asked me was interesting because it required to formulate a data structure to solve a problem in the real world. Tracking browser history was something I had never considered, but I was required to implement a back-end API front-end developers could use when tracking a user's history in a browser. I had to define an interface, come up with tests and edge cases, and optimize the solution when there were more requirements to be met.

The second question was a string parsing problem. Given a string, I had to print out all the patterns of A's and B's. So, for instance, a string $AaheaabelloaaabB$ would print out $Aa$, $aa$, $b$, $aaa$, and $bB$.

## Post Interviews

Because all my interviews were technical and I wasn't given a single behavioral or product design question, I was upset with how the interviews went. Every single question I was asked was a question that I had never encountered before.

In retrospect, this was an important skill to learn because there will always be a technical question that is outside the scope of what one has prepared for. It is a very useful skill to understand the problem, come up with different solutions, and then optimize them.

Regardless of how the interviews went, we were given a free jacket afterwards and $55 to sightsee Seattle. I had a couple of friends at the University of Washington, so I went to go visit them. Later that night, we enjoyed a nice Thai meal on Microsoft's expenses!

Overall, the experience was great. It was my first time being blown out to an on-site interview. I had a great time speaking with other candidates. They were all over the nation - from Harvard to Iowa State.

## Decision

It took about a week for the recruiter to get back to me with a decision. I was getting very nervous during this time because a girl I had met from Harvard during the interviews had already gotten a response a couple of days after the interview. On Quora, I had read that if Microsoft takes over a week to get back to candidates, those candidates are on the "maybe" pile.

Fortunately, the recruiter contacted me on the Wednesday after with an offer! I was ecstatic and shouted a couple of times in my dorm room before settling down. Based on my interview performance and interests, I was placed in the Cloud Enterprise and Management division, which is exactly where I wanted to work. Microsoft has been doing really well with Azure lately - because I worked with AWS at my previous internship, I wanted to get experience with Azure.

I hope this article shed some light on what the Microsoft interviews are like and what to expect. In terms of the Explorer interviews themselves, I eventually figured that the questions interviewers asked are tailored to each candidates' experiences and resume. It should not have come as a surprise that I didn't get a basic array reversal question.

Microsoft doesn't look for candidates that merely know things, but rather looks for a potential to grow. The best interview prep one can do is do a combination of technical and product design questions that he or she has never seen before. Catching someone off-guard is probably the best way to evaluate their thought process. Anyways, good luck for your interviews and find me on social media if you have any questions!
