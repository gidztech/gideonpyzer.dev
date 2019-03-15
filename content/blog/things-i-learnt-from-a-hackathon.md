+++
title = "Thing's I learnt from a hackathon"
description = "I took park in a hackathon this month at work, my first one. 3 days are set aside each year for running the hackathon across the company. The idea is to experiment with new technology and see what innovative solutions we can come up with."
tags = [
    "AWS",
    "hackathon",
    "appsync",
    "graphql",
    "apollo",
    "react",
    "prosemirror"
]
image = "hackathon/coding.jpg"
date = "2017-12-26"
highlight = "true"
aliases = [
    "/blog/things-i-learnt-from-a-hackathon"
]
+++

![Hackathon](/img/blog/hackathon/coding.jpg)

I took park in a hackathon this month at work, my first one. 3 days are set aside each year for running the hackathon across the company. The idea is to experiment with new technology and see what innovative solutions we can come up with.

This year the theme was AWS. We recently migrated our US app instance to AWS, but only a small proportion of the product engineering team had much experience with. The themed hackathon was an opportunity for all of us to get some AWS experience, and see how we could use some of the hundreds of services available on the platform. 

# My team
There were only 3 of us in my team. This was the smallest of the 6 teams taking part - 2 developers, 1 sales consultant. 

# Our hack
Our hack was a collaborative note editor. We already have a standard note editor feature in the app, as well as office-online integration for collaborative editing on Microsoft Office documents. 

However, we wanted to provide the capability to simpler documents, typically plain text or markdown. This would also allow customers who don't have office online subscriptions to use something less expensive.

# The tech
- AWS AppSync
- Apollo GraphQL
- React Apollo
- React
- Prosemirror

## Some details
*My understanding of AppSync is a little sparse, having only spent 2 days hacking around with it, but I'll give it a go.*

AWS recently announced a service called AppSync at the [re:Invent 2017](https://reinvent.awsevents.com/) conference in Las Vegas.

>> AWS AppSync automatically updates the data in web and mobile applications in real time, and updates data for offline users as soon as they reconnect. AppSync makes it easy to build collaborative mobile and web applications that deliver responsive, collaborative user experiences. 
>> Source: [AWS AppSync](https://aws.amazon.com/appsync/)

This sounded perfect for what we were trying to achieve. We were fortunate enough to get access to the public preview program, allowing us to use it in the hackathon.

AppSync uses the GraphQL standard to fetch and modify data in a data store, such as a DynamoDB database. Bindings are set up between the GraphQL data schema and the database using resolver functions in the AppSync web console. 

GraphQL is really cool. It's similar to REST, but a major difference is the schema of the data is typed and not coupled to the way you fetch the data. The client will only ask for specific fields to be returned by the server, not the whole set, which is really efficient. You can also request from multiple resources in a single request. Check out the [GraphQL](http://graphql.org/) website for more details. AppSync uses some of [Apollo GraphQL](https://www.apollographql.com/) to implement the standard.

The AppSync client side library is a wrapper that deals with authentication with the AppSync service, includes Apollo's client side library, and sets up real-time subscriptions using an MQTT library over WebSockets. The details are generally abstracted away.

In order to implement the collaborative not editor, we used the [Prosemirror](https://prosemirror.net/) library. It has collaborative editing capabilities built in, but there's a bit of work to wire it up. 

# The result
We were able to get something demoable by the end of the hackathon, but we had some problems along the way, which prevented us from completing it.

![AWSome Note](/img/blog/hackathon/editing.gif "AWSome Note")

# The problems
I think the main issue was the amount of new tech we had to learn and use in such a short space of time. Every single part of the tech stack was new, even React to an extent, which I have a little experience with. 

## Problem 1
In the days leading up to the hackathon, we were able to discuss ideas and tech we could use for the hackathon. However, we only got access to AppSync the day before. We came up with alternative solutions in the meantime, such as using PouchDB on the client with SocketIO for real-time updates, but these wouldn't have used AWS for more than infrastructure.

We were able to look a little into GraphQL the night before, but the rest of the tech was going to be completely new for day 1.

## Problem 2
As a result of problem 1, we ending up taking the [AppSync sample client project](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react) and building on top of that. I spent the morning of day 1 building out our own GraphQL schema and hooking it up in the client library. It was going well until we found that the data was not syncing. The subscription handlers were not firing after mutations. My colleague and I spent hours trying to debug it with little luck. We ended up starting over again, but instead of writing our own schema, we used the sample one. The schema difference is "documents and deltas" vs "events and comments", but it worked. All of that took us to the end of day 1.

## Problem 3
The collaborative editing in Prosemirror was a little tricky to understand at a glance. My colleague spent some time going through the example, and then simplified it so we could get something done quickly. However, by simplifying it, we messed up some of the synchronisation mechanisms. We ended up with the client sending deltas (document changes) to GraphQL, then each other client would get the delta (via subscription) and apply the change on top of their document. It worked if you typed slowly, but once you typed normally, it would all get out of sync. 

We needed a central place to consolidate the deltas and send them off the client, in the right order. A lambda may have helped with that.

## Problem 4
We originally planned to indicate in the UI who was typing in the editor, but because of all our tech problems, we never got around to it. This made the demo less impressive and rather confusing, because you had no idea who was typing what.

# The winner
The winning team primarily used [Amazon Rekognition](https://aws.amazon.com/rekognition/) (yes, I thought it was a typo too) for tagging images on upload to our system, allowing really powerful search capabilities in the application. 

The simplicity of the architecture and the services used is one of main reasons it won. Essentially, an image is uploaded to an API and the tags are returned. I believe they used API Gateway and a Lambda as a proxy for the requests. This allowed the team to concentrate more on the user interface, which is an existing page in our application. They were able to use our current data-binding library and services, which meant less time was needed learning and more time spent coding. 

# Conclusion
The hackathon was fun and I learnt a fair bit, but it rather stressful at times when things didn't work. AppSync is really nice, but a lot of the abstractions on the client-side libraries (React Apollo particularly) made it difficult to pick things up quickly and debug problems. There's a lot of new concepts, and building them up slowly would have yielded much better results, though that is not really an option in a hackathon. 

This hackathon has taught me to keep the idea and architecture simple, and build on it if time permits.
