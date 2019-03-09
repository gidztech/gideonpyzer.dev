+++
title = "Documenting code"
description = "Documenting your code is vital when working on a project with multiple developers. But what is the correct level of documentation?"
tags = [
    "documentation",
    "code",
    "comments",
]
date = "2019-03-09"
image = "documentation/documentation.jpg"
highlight = "true"
+++

<img src="/img/blog/documentation/documentation.jpg" style="width: 600px; margin-top: 20px; border:0" />

# Introduction
Documenting your code is vital when working on a project involving multiple developers. If you join a new team, you would hope that their work is in a state that you could fairly comfortably pick up and roll with. People should strive to code with empathy for the current and future team.

The level of documentation you provide, and the format of it, depends on context. I'm not soley referring to comments in code, but also ticekts, commit messages, READMEs, manuals etc. 

I've worked on projects where there's been no comments, or close to none. This in itself doesn't necessarily mean the project is in a bad state, but it can be an indication that future developers on the project may be at a disadvantage.

Equally, I've seen code where there's endless comments describing everything that has been done, which is a lot of noise. When I was in my first year of university, I found myself commenting on almost every line of code I wrote. I cringe at the thought nowadays, but perhaps at the time it wasn't a bad idea, since it proveded to the professor that I actually understand what I was doing. I didn't just copy and paste code from StackOverflow, that I'm sure others probably did! 😇

# When should I comment?
My view is that:

> Code is the what, comments are the why

Code should be as descriptive as possible, telling the other developers what it's doing without having to read it as if they were a compiler. Comments should be used to document decisions that have been made. We will see shortly that there's some exceptions to this.

## Code should be self documenting
Code is a set of instructions for the computer to perform. In higher level languages, it becomes more descriptive and verbose. For example, `while` loops and `if/else` blocks can typically be understood by anyone. Code pretty much tells a story. This is particulrly the case for imperitive coding styles, but functional, declarative and object-oriented paradigms correspond to real world concepts.

Function and variable names are chosen by the developer, and make up the majority of the code you write or read. That means, it's important that you use meaningful names. Variable names, such as `x` or `i` are not useful in understanding the application. They should generally be avoided. Now, you may choose to use `i` for iterators, provided they are boxed up inside a single-purpose function that has a name that describes what it does. That way, the use doesn't even need to understand the internals of it. 

For the most part, with good descriptive names, your code should be largely understandable by other developers. If another developer can't understand the code from reading it, it's a sign that the code is overly complex, badly organised, and needs to be refactored.

The main exception for me is Regular Expressions (Regex). Now these beauties are incredibly powerful, and I'd struggle if they didn't exist, but oh boy are they difficult to read. Sure, if you have a `\d+` matcher, I wouldn't necessarily feel the need to comment, especially if your function name is `getIdFromWorkspace`. But if you have something more complex, a function name is going to be difficult to write to the required level of detail. This is where a comment goes a long way. It's ESSENTIAL! 

Let's just say, I've seen some crazy Regex around with no comments, and I've wanted to cry.


## Documenting code decisions
How do you document decisions you've made in your code? Sometimes you may read some code and wonder why the developer did it this way. This is partcularly the case where the developer has done something a little different from the norm. Of course, sometimes doing something agains the norm is a red flag, but in other cases, there may be legitimite reasons. 

For example, you may have made the decision not to optimize some code because it's not going to scale. You might warn a future developer not to copy and paste this code for other features because of the limitation. 

I like to write comments in code where I've made a decision like this. Some people may argue that comments become out of date, but so does code. If your code is going to change, the comments that decorate it will change or be removed anyway. 

However, the most effective way is to put a short comment and link to a ticket (e.g. Jira or Github Issue), where you can put a lot more detail. You should also always try to put the ticket number in your commit message. That way, the developer can see a full discussion on the decisions that have been made.

I can't count the number of times I've struggled to work out why something was done the way it was, and not had any way to find out more. What if the original developer has left the company? All those business decisions are lost. 

### Formal documentation
Documentation in code or tickets is great, but for more visibility, I'd document bigger decisions somewhere more central, and in particular, accessible by product owners and other stakeholders. If you're using Atlassian products, Confluence is quite a good choice for this. We also use our own product, Huddle, to store project documents. Since we are a secure collaboration application, it's very fitting. 

MORE TO DO HERE 



## TODOs
- TODOs are good in principle and IDEs and editors have some nice plugins for viewing them in panels. The problem is they often remain there for years. Use them for very temporary work, e.g. for a particular PR you’re working on, but if you’re likely to leave it in, don’t. Create a story and put it on the backlog with detail, or solve the problem straight away if applicable.

