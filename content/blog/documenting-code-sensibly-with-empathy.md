+++
title = "Documenting code sensibly with empathy"
description = "Documenting your code is vital when working on a project with multiple developers. But what is the correct level of documentation?"
tags = [
    "documentation",
    "code",
    "comments",
    "empathy",
    "ADR",
    "tests"
]
date = "2019-03-11"
image = "documentation/documentation.jpg"
highlight = "true"
+++

# 🏃‍♂️ Motivation
Documenting your code is vital when working on a project involving a whole team. When you join a new team, you would hope that the work is in a state that would allow you to fairly comfortably pick up and roll with. As such, **people should always strive to code with empathy for the other people on the team, both current and future.**

<img src="/img/blog/documentation/i-know-that-feel-bro.jpg" style="width: 300px; margin-top: 20px; border:0" />

The level of detail and format of the documentation you provide is context dependent. These may include comments in code, tickets, commit messages, READMEs, ADRs, etc.

I've worked on codebases where there's been no comments, or close to none. This in itself doesn't necessarily mean the project is in a bad state, but it can be an indication that future developers on the project may be at a disadvantage.

Equally, I've seen code where there's endless comments describing everything that has been done, which is a lot of noise. When I was in my first year of university, I found myself commenting on almost every line of code I wrote. I cringe at the thought nowadays! 🙀


<img src="/img/blog/documentation/code.jpg" style="width: 600px; margin-top: 20px; border:0" />

# When should I comment?
My basic view is that:

> Code is the what, comments are the why

Code should be as descriptive as possible, telling the other developers what it's doing without them having to read it as if they were a compiler. Comments should be used to document noteworthy decisions that have been made. We will see shortly that there's some exceptions.

## ⌨️ Code should be self-documenting
Code is a set of instructions for the computer to perform. In higher level languages, it becomes more descriptive and verbose. For example, `while` loops and `if/else` blocks can typically be understood by anyone. Code pretty much tells a story, particularly for imperative coding styles. That said, most paradigms correspond to real world concepts.

Function and variable names are chosen by the developer, and make up the majority of the code you read or write. That means, it's important that you use meaningful names. For example, I avoid using variable names like `count` or `x`. The former is ambiguous, why not `numOfUsers` instead? `x` means absolutely nothing. That said, you may choose to use `i` for an iterator inside a single-purpose function, where the value of `i` itself has little meaning in itself.

For the most part, with good descriptive names, your code should be largely understandable by other developers. If another developer can't understand the code from reading it, it's a sign that the code is overly complex, badly organised, and needs to be refactored.

### 😭 Regular Expressions
This is one of a handful of possible exceptions. Now these beauties are incredibly powerful, and I couldn't imagine a world without them, but oh boy are they difficult to read. Sure, if you have a `\d+` matcher, I wouldn't necessarily feel the need to comment, especially if your function name is something like `getIdFromWorkspace`. 

But if you have something more complex, a decent descriptive function name is going to be difficult to come up with. This is where a comment goes a long way. In fact, it's **ESSENTIAL**! 

Let's just say, I've seen some crazy regex around with no comments, and I've wanted to cry! 😭

```js
/=(?:[^\[\|]*?(?:\[\[[^\|\]]*(?:\|(?:[^\|\]]*))?\]\])?)+(?:\||\}\})/
```

## 📝 Documenting code decisions
How do you document decisions you've made in your code? You might read some code and wonder why the developer did it a particular way. This is particularly the case where the developer has done something a little different from the norm. Of course, sometimes doing something against the norm is a red flag, but in other cases, there may be legitimate reasons. 

For example, you may have made the decision not to optimize some code because there's no need for it to scale. You might warn a future developer not to copy and paste this code for other features because of that limitation. Developers are lazy, copy and paste all the things.

I like to write comments in code where I've made a decision like this. Some people will argue that comments become out of date, but so does code. If your code is going to change, the comments that decorate it will likely change or be removed anyway. It's not a reason to avoid comments.

However, the most effective way is to put a short comment and link to a ticket (e.g. Jira or Github Issue), where you can put a lot more detail. You should also always try to put the ticket number in your commit message. That way, the developer can see the full discussion.

I can't count the number of times I've struggled to work out why something was done the way it was, and not had any way to find out more. What if the original developer has left the company? All those business decisions are lost. **Be an empathetic developer!**

### 👔 Formal documentation
Comments in code and discussions in tickets are great, but for more visibility on larger decisions, it's best to document them somewhere more central, and in particular, accessible by product owners and other main stakeholders. 

If you're using Atlassian products, Confluence is quite a good choice for this. We also use our own product, [Huddle](https://www.huddle.com/), to store project documents. Since we are a secure team collaboration application, it's the perfect choice for us.

#### Architecture Decision Records (ADRs)
As part of our software development lifecycle, we document project decisions in an [Architecture Decision Records (ADR)](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions). At the start of the project, we review the requirements and possible solutions to achieve the end goal, using [ATAM](https://en.wikipedia.org/wiki/Architecture_tradeoff_analysis_method). We may consider things like security, performance, testability. maintainability, etc. The ADR allows us to document what we have considered, and the final decision we made. It's a living document, so we can update it throughout the project. 

This document is really useful because I can look up past features we've implemented and identity why we did something a particular way, and see the alternatives we considered.

## Tests provide requirements documentation
You can understand a lot about an application from looking at tests cases. They're usually written in BDD or TDD form, like `describe` and `it` blocks, so they are easy to read. The implementation details are blackboxed because all we see are the input and outputs. Once we understand that, when we need to update the code, we already have some context.


# 🥁 Conclusion
As we've seen, there are many ways to effectively document your code. Where possible, code should be self-documenting, but in a few cases, comments are definitely required. Noteworthy decisions may be documented as comments in code, with deeper discussions linked to via a ticket reference. For larger architectural decisions, storing them centrally provides greater visibility. Tests provide an understanding of the application's requirements. 

By following sensible practices, we can help other members of the team, current and future, understand the code we've written, and the reasons behind decisions that were made. 

**Code with empathy, don't make me cry like regex does!** 🤗