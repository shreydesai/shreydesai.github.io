---
layout: post
title: "Resolving Merge Conflicts in Git"
date: 2016-06-09 05:25:00
categories: Git
featured_image: /images/cover.jpg
---

Wrapping up my first week at [Sizzle](http://gosizzle.io), I have learned some important concepts in code collaboration. Putting this into perspective, the internship is remote; this makes it especially important to track, document, and version the code. We have been using Git as our version control system and recently I ran into some troubles merging my code with the existing codebase – a daunting task for newcomers. Luckily, I figured out how to properly use `git merge`, ensuring my code would not break anything in the original repository.

## The Git Model

As I've been working with Git for a couple of months now, I can tell it is a powerful version control system. It especially has a unique way of tracking existing changes and pushing files to [GitHub](https://github.com). With a lot of the work happening in the background, it was hard for me to really understand what was going on, so solving Git problems required a deep understanding of how the software actually operated.

<img src="http://i.imgur.com/ID8eN0U.png" width="40%">

The Git system can be aptly represented by a linear model, where the blue circles are commits – these can come from different machines, assuming the code does not conflict (I'll get to that later). `develop` on the bottom represents the name of the branch and `HEAD` is the current state of the repository.

## Sizzle's Repository

Sizzle's repository is broken up into two main branches – `master` and `develop`. The `master` branch is standard; most repositories will use this one as the "main" one, with all of the tested and documented code. The `develop` branch is specifically used for development. Every week, the contents of the `develop` branch is merged into the `master` branch and thus released into production.

This week, I have been assigned a couple of bugs that disrupt user experience in Sizzle's main product. For each bug, my mentor told me to branch off of the `develop` branch and commit all of my fixes to the particular branch. I would then send in a pull request and assuming all of the bugs were squashed, the code would be merged into the `develop` branch.

<img src="http://i.imgur.com/nBZtnA8.png" width="50%">

I was mostly working with a JavaScript file, which controlled the behavior of Polymer web elements and the general website, mostly written on top of jQuery. Let's call it `script.js` for the purpose of this article. Sizzle's build process also consists of minifying the files, so once my changes to `script.js` were complete, I would run a script to update `script.min.js`. Over time, the commits in my `bug/1523-modal` branch accumulated.

<img src="http://i.imgur.com/DcuEqMQ.png" width="50%">

## Merge Conflicts

Here's where things got a bit annoying. At the same time I was updating the `script.min.js` files, my mentor was doing the same, leaving us with completely different `script.min.js` files when comparing the two. Of course, he was the administrator of the `develop` branch; this means he could commit directly to it while I had to go through the pain of maintaining my separate branch.

When it finally came time to commit my changes with the `develop` branch, I was inundated with a ton of merge conflicts because we were both modifying the same file. My initial reaction was to run `git rebase develop`, which rebased the `bug/1523-modal` branch with respect to the `develop` branch. This did not turn out so well because I was injected into a long list of conflicts and in a nutshell, I did not know what I was doing.

With some guidance from the Internet and a lot of shell inputs later, I finally figured out how to use the `git merge` command, which I now know has some crucial differences with `git rebase`. `git merge`, just like `git rebase`, listed out the different files that were conflicting, but it was optimal in that it provided me a clean way to resolve the issues.

Looking back at the `script.min.js` file, I now saw something like this:

```javascript
<<<<<<< HEAD
// new code + bug changes
=======
// old code
>>>>>>> 1fb7853
```

Yes, it looked scary at first, but I realized it was not. All I had to do was preserve my changes and remove the old, so essentially remove everything below the `=======` and up to `>>>>>>>`. And, of course, I had to remove `<<<<<<< HEAD` so my JavaScript would compile correctly. The rest of the process was easy – all I had to do was `git add .` and `git commit -m ""` and push the changes to the `develop` branch. The sight of "Can automatically merge" was the sweet smell of success.

I hope to write more articles regarding my experience during this summer internship. Next week, I should be working more heavily with Polymer web elements and learning more advanced topics in jQuery, namely AJAX. I'll be sure to shed my insight on them once I learn more.
