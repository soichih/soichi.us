---
title: "Coding Style"
date: 2020-08-01T18:26:57-04:00
draft: true
---

Here is the summary of my favorite coding style. I hope I can use this to discuss individual principles with other developers and/or share this with junior developers to bring them up-to-speed.

## 1. Always allow for exceptions to any coding style

I'd like to start out by making it clear that there isn't any coding style / principle that I obey at all time. This is why I don't really like project that enforces any linters to the point that you can not even commit / continue with CI process unless you *fix* all style guide. 

## 2. Code is for human, not for computers.

We shouldn't be making our code easier to read by computers. 

## 3. Don't fix bugs, remove bad code.

Fix bug by removing code that is creating the bug - rather than adding more code to treats the symptoms of the bug.

## 4. "You aren't gonna need it" (YAGNI)

Don't implement functionality that you think you probably going to need it. Wait until you actually need it.

## 5. Writing twice is too many (DRY)

If there is a similar code repeated *twice* anywhere in your code, refactor them immediately. Don't want until you have 3 instances of similar code. 

# Non-coding related principle

## 1. Don't blame the user when your test case is incomplete

When someone sends you a bug report, but you can't reproduce it on your dev environment, it usually means that your test-case is failing to identify a bug that your users are facing.