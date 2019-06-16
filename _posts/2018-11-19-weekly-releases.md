---
layout: post
title: Weekly Releases
categories: []
tags: [Product Management]
---

We've recently got back into the habit of weekly releases in preparation for a 3.0.0 release of [Enact](https://www.enactjs.com/) and it has reminded how useful they are. They add some much needed structure and predictability which help us ship a consistently high quality product.

> For app teams, weekly releases may seem silly if you are in the habit of daily releases. As a framework team, we want to be more measured with releases, particularly minor and major releases, which might introduce unnecessary churn into our users' code base as they try to adopt the latest. I've started coalescing some thoughts in a draft post so more to come if you're interested in this topic.

# Challenges with ad hoc releases 

Before we started with weekly releases, our release schedule was driven primarily by customer need. There is nothing inherently wrong with this and doing so encourages alignment with your users which is a great thing. In our case, we had one significant customer driving the schedule and, by extension, the content of each release. As a result, we found it difficult to prioritize technical debt paydown and new feature work that didn't directly map to that customer's needs.

This process also delayed shipping features and fixes to any customers. We land new features and fixes daily but if the customer schedules didn't require a release for a month, there could be useful code sitting idle for a long time. Further, the longer it takes code to get in the hands of users, the longer the feedback loop on the suitability of the change.

One of more nefarious challeges with ad hoc, customer-driven releases was Fear of Missing Out (FOMO)! As the release date drew closer -- and particularly during the last day or two during our final test pass -- more features and fixes would sneak their way into release. We would find reasonable ways to justify it:

* This is a "low risk" change so we should just drop it in.
* This was reported last night so wouldn't it be great if we could include it in the release?
* This feature isn't a high priority but it would be too bad if our customers had to wait another month to start using it so let's slide it in.
* Well, we added this feature 2 weeks ago but just now discovered this issue with it. We could revert it but X, Y, or Z was built on it so let's put a fix in the moment before we release and hope that everything goes okay.

The last one might not be reasonable but that sort of rationalization happens when you lack structure in your releases. Our heart was in the right place -- we wanted to add value for our customers -- but it caused plenty of last minute scrambling and often impacted quality.

# Weekly releases to the rescue!

After having muddled through ad hoc releases for a while, we decided to move to a more regular cadence of weekly releases on Monday afternoons and it has been wonderful! It doesn't come without costs and we still "sneak in" the occasional last minute fix but the structure has improved our team health and hopefully helped our customers with the predictability.

Foremost, features and fixes land sooner for customers. We still try to batch up new features into less frequent API changes but bug fixes land much sooner. Weekly releases have helped us balance requirements for new products and customers which can adopt the more frequent releases based on their own timelines. We're also able to tackle technical debt incrementally and validate the changes through our release testing and from customer feedback much more quickly. Ultimately, we're now in more control of what and when is included in a release which allows us to practice product management rather than fighting fires.

We've curtailed FOMO because we have a known next release in which changes can land if they aren't ready. With a shorter release cycle, we're less likely to have compounded changes so reverting troublesome pull requests is less daunting. We also maintain the ability to do mid-week hot fixes if a critical issue arises that must be addressed.

Release management became more predictable for everyone involved. It's allowed us to define repeatable processes around when we accept work for a release, when integration testing occurs, and when releases are tagged and shared with our customers. We aim to create our release branch (we use [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/) for release management and have found it to be a good branch management strategy) with all of the changes for the release on Friday afternoon. That gives us the afternoon to define a test strategy. On Monday morning, we run our manual and automated tests, triage any issues discovered, and if all has gone well, release later that day!

---



_I'm curious how others have tackled this problem, particularly if you're in the business of maintaining APIs. What are other approaches to do this better? Reach out via a [Twitter](https://twitter.com/theryanjduffy), [email](ryan@tiqtech.com), or [GitHub Issue](https://github.com/ryanjduffy/blog/issues)._
