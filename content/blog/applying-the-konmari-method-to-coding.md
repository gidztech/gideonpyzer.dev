+++
title = "Applying the KonMari method to coding"
description = "Does this code spark joy? While I was watching \"Tidying Up with Marie Kondo\", I thought about how we could apply her methods to writing and maintaining code."
tags = [
    "konmari",
    "marie kondo",
    "code",
    "refactoring",
]
date = "2019-01-31"
image = "tidying/mess.jpg"
highlight = "true"
+++

<img src="/img/blog/tidying/mess.jpg" style="width: 600px; margin-top: 20px; border:0" />

If you haven't heard of Marie Kondo, you must be living in a bubble. Her recent show on Netflix, "Tidying Up with Marie Kondo" has been extremely popular, and sparked a lot of discussion online.

The show involves her going to other people's homes, mainly in California, and helping families clear and organize their belonging. She invented the KonMari method for achieving this.

While I was watching the show, I thought about how we could apply her methods to writing and maintaining code. I think the same principles apply.

> Does this code spark joy?

Over time, projects can build up a lot of dead code, or the quality of it starts to suffer. When you read a piece of code in the project you are working on, ask yourself whether you're happy with it. If it makes you feel uncomfortable, there is probably something wrong with it. Trust your instinct.

Since the code is most likely shared among other people, consider whether others will be happy with it too. Does it read well? Is it succinct? Will they be able to understand it quickly?

# Clear out code you don't need

Some people have a habit of hoarding items. I've been guilty of that a few times before. You hold onto items that you MIGHT need or want in the future, but most of the time you never look at them again. This not only happens with tangible items, but also with code.

Say that you created a component a while back, but then you ended up using another one later on. You might have decided to keep the original component just in case you require it in the future. This can lead to codebases eventually becoming out of control. Just thank it, and chuck it out.

You have a couple of options in the rare case that you need it again:

1. **Write it again** - You'll code it in half the time, and it's likely to be better quality
2. **Trust source control** - It's all in Git. You can restore it. It's a shame this doesn't apply to life

If you come across code that you have become quite attached too, leave it until the end. Once you've practiced removing other code, it will become easier to detach yourself. (sentimentals)

Once you've done all of this, tidying up the project becomes a lot more manageable.

# Compartmentalize

Marie always brings small boxes, of varying sizes, for her clients to use for storing items. By compartmentalizing things, it becomes much easier to find and keep them tidy. You can also move whole compartments around with less mental effort.

Coding can benefit from the same mechanism. Splitting large functions or classes into smaller, single purpose ones, makes it easier to reason with.

You can organize your code into categories. For example, pages and components, and each component can have a folder. You know exactly where everything is.

# Final thoughts

Marie found that her client's relationships improved when they were able to clear out and tidy their belongings. I think tensions between developers on projects could massively be reduced by applying these techniques. More time can be spent building great features instead of spending hours searching for 1 of 8 similar, but slightly different, modal components in your codebase!
