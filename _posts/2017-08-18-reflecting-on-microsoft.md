---
layout: post
title: "Reflecting on Microsoft"
date: 2017-08-18
comments: true
---

## Introduction

Today marked my last day interning at Microsoft. It was bittersweet -- I spent the last 12 weeks contributing to a wonderful product, meeting extremely intelligent people, and immersing myself into Microsoft culture.

I interned on the [Azure Stack](https://azure.microsoft.com/en-us/overview/azure-stack/) team, a private cloud platform that extended Azure resources on-premises. The product targeted companies with latency issues, special regulatory requirements, and existing on-premise data centers. Because the product reached general availability during my stay, it was fascinating to witness its development and social media coverage.

My project was to build an internal development tool for the Azure Stack team. Each individual task that was completed during deployment was defined in a configuration file. Many tasks were dependent on other tasks, forming a [directed graph](https://en.wikipedia.org/wiki/Directed_graph) structure. For example, services to be installed on a virtual machine required the virtual machine to be started successfully. Because Azure Stack was an extension of Azure, deployment consisted of _hundreds_ of interconnected tasks.

Not surprisingly, the configuration file that kept track of deployment steps was thousands of lines long. Editing the file -- let alone understanding it conceptually -- was extremely difficult. I collaborated with two other interns to create a command line interface (CLI) that a) generated graph-based visualizations of configuration files, b) presented a visual [diff](https://en.wikipedia.org/wiki/Diff_utility) of two versions of a configuration file, and c) integrated with log data to display important deployment analytics.

We used C# for development, [Powershell](https://en.wikipedia.org/wiki/PowerShell) for scripting, and [DGML](https://en.wikipedia.org/wiki/DGML) for directed graph rendering. Most importantly, this project taught me a lot about object-oriented design, advanced version control with Git, and the importance of continuous integration.

The [Explorer program](https://careers.microsoft.com/students/explore) is unique in that it exposes students to the fields of software engineering and program management. In the beginning of the summer, I was dead set on becoming a developer and wasn't interested in learning about the business side of things. However, I kept an open mind, ultimately learning important lessons from both fields. I could go on about C# language internals, Git merging vs. rebasing, or concepts in graph theory, but instead I want to talk about some _fundamental_ principles.

## Lesson 1 -- Customer Obsession

Engineers don't ask the question "So what?" enough.

When our manager first pitched our project idea, it was extremely vague. "Visualizing configuration files" didn't really mean anything without a proper context. _Who was visualizing it? What was being visualized? How would it be visualized?_

As a program manager, our goal was to interview our customers -- Azure Stack developers -- to get the necessary context. We needed to understand the problems of the status quo, discuss trade offs between various solutions, and build a set of features for the final product. The goal was to put the customer _first_ because the customer was the end user. It was meaningless to inject personal bias and opinions in the very start because it wouldn't reflect the needs and requirements of the customer.

Developers are usually not required to go through the aforementioned steps. They often receive a document that specs out exactly what needs to be done. This eliminates most ambiguities -- UML class diagrams and mockups have all the necessary information.

However, the customer is still a paramount concern when considering issues like software usage and maintainability. It would be disastrous if an engineer checked in convoluted code that was undecipherable by his or her peers. These sorts of practices would most likely impede the progress of development, resulting in company stagnation and decline over time.

Developers should be *customer obsessed*. As software engineers, we should constantly consider whether our code is meaningful, resilient, and impactful. Even though our job doesn't require us to directly reach out to the end users, the architecture and functionality of the product is in our hands. It's important not to lose yourself in the technical details -- consider the bigger picture and _always_ ask questions like who, what, why and how.

## Lesson 2 -- Programming is Easy

Given experience with various programming languages and frameworks, any developer can throw together code for a product. Most projects follow common design patterns, for instance, distributed applications have client-server models and single page web applications follow MVC patterns.

Our team's project was no different -- the prototypes for the static analysis tools were completed in two days. If you went to Visual Studio and clicked "Start," the project would successfully build and run. The entire CLI was packaged into two files, each with one method. The core features had been implemented but performance and maintainability were not taken into account.

Unfortunately, it would only get worse. Over the next couple of weeks, we needed to add more and more features to the CLI, which made the _one_ method in each file grow exponentially. The methods quickly spanned over 500 SLOC -- they were completely unmaintainable. A quick glance at the project revealed massive blocks of convoluted logic, nested `for` and `while` loops, and an abuse of `if`/`else` statements.

We decided to refactor the project -- breaking down the monolithic methods into subclasses, taking advantage of inheritance and polymorphism, and adopting common design patterns. While this took a lot of time and effort, the project became significantly easier to unit test, error handling was intuitive, and catching bugs was easy.

Software engineering is 10% programming and 90% maintaining -- or something along those lines. The code that you write doesn't belong to you, but rather, it belongs to everyone. Chances are that a developer's contributions to a codebase will affect hundreds or thousands of other developers. As a company evolves, its requirements, people, and software will change as well.

This means that the barrier to entry must remain important during the development process. Nobody likes writing documentation, but everyone loves documented code. Nobody likes testing their code, but everyone loves tested code. Nobody likes refactoring, but everyone loves refactored code. Development is full of these ironies.

It's easy to program, but it's hard to refactor. Take the extra step to revisit your code, making it as performant, understandable, and maintainable as possible. This will ensure your work will be impactful for years to come.

## Lesson 3 -- Testing-Driven Development

Prior to this summer, I had completely confidence in my code. For the most part, my scripts and projects were functionally correct, but there were occasional blunders. It didn't really matter if there was no backwards compatibility, the build sporadically failed, or bugs were present. Nobody was depending on my projects, so correctness was not a priority.

However, my mindset had to change when we began developing the static analysis tools. The purpose of our project was to _increase_ Azure Stack developer productivity, so if our tools randomly failed, developers would spend time debugging our code instead of doing actual work. This pressure didn't mean we had to write _more_ tests, but rather embrace a different methodology -- test-driven development.

At the core, test-driven development emphasizes testing before developing. The process includes creating unit and integration tests, writing code, and ensuring the code passes the tests. It sounds easy, but it's actually not -- writing tests is yet another time investment. So, is it worth it?

First, testing gives you a better understanding of your product. Questions such as _What should the program do?_, _What are the expected results?_, and _Does this code handle all possible cases?_ are all answered by tests. Unit tests should be a verification tool that the product meets all guidelines and criteria. Integration tests should ensure that different, isolated components of a product are compatible with each other. Testing establishes confidence that the product's behavior is deterministic, leading to a positive user experience.

Second, when inputs and requirements change, tests verify whether a program still works. For example, our project performs static analysis on configuration files, but the structure of these configuration files is always subject to change. Given these changes, there are aspects of our project that will break. Tests would allow developers to pinpoint exactly where refactoring is needed.

Third, tests provide a breadth _and_ depth of understanding. Engineers can write hundreds of tests and each test can verify an innumerable amount of details. When projects are small, it's possible to manually test small components with quick `print` statements, but this strategy does not scale. For example, an HTTP request has authorization headers, type of method, the actual payload, and so much more. It would be impossible to manually test thousands of API calls, but tests could do this in an automated fashion.

Testing sounds wonderful, but it does come with baggage. It takes creativity to come up with tests, refactoring code requires refactoring tests as well, and the test-driven development cycle never stops. But what's the alternative -- no tests? When you work for someone and you work at scale, functional correctness is a must. Test-driven development might be an obstacle now, but it's a necessary investment in the success and productivity of any engineer.

## Conclusion

I highly recommend working at Microsoft, whether you are a university intern or full-time employee. The principles and methodologies I learned were invaluable -- they will set the foundation for my future endeavors in software engineering. This company is unique in that it has many successful, innovative products -- Windows, Xbox, HoloLens, Azure, etc. Each product contains hundreds of teams, each with a unique culture and dynamic. As a software engineer or program manager, you will work on exciting, fast-paced projects and impact billions of people in the process.

If you have any questions about the Microsoft intern program, don't hesitate to reach out to me on social media. I'm always excited to hear about people's experiences and offer advice if needed.
