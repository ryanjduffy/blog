---
title: What Makes a Good Pull Request?
layout: post
---

I get to spend a lot of time reviewing other engineers' code. Over time, I've
found that it has helped me become a better engineer as I learned new techniques
to solve problems, explored parts of the code base that I haven't worked in
before, and improved my ability to communicate effectively to others.

Through the course of my reviews, I've seen some common practices that I feel
make for good pull requests. Some may be obvious but others are a bit more
subtle.

<!--more-->

# Respect the Codebase

As a "guest" (or even as a maintainer, in my opinion), a bug fix or a feature
implementation is not the time to introduce new patterns or conventions. It's
also not the time to debate space vs tabs, ternaries vs `if` blocks vs. `switch`
statements vs. logical operators, or the merits of functional programming and
the pitfalls of prototypical inheritance.

Respect the current codebase and follow its conventions. Deviate only when
necessary and only after clearly describing why. If you're contributing to an
open source project, the maintainer has enough on their plate. Focus on solving
the problem and avoid convention controversy. It will help everyone move
forward.

# Solve One Problem

In addition to avoiding convention debates, try to limit the scope of your
change to a single problem. Smaller change sets are easier to review and
therefore easier to land. The more code you touch, the more risk you introduce
in the process. Is that removed code really cruft? Does the refactoring conflict
with other in flight work of which you weren't aware. This one can be a bit more
art than science but when in doubt, favor less change.

> However, Pull Requests are generally "free" so don't limit yourself to one! If
> you have housekeeping or refactoring to suggest, those changes are often
> welcome too. Open a second pull request with the changes and move them along
> separately. This helps the work move independently and often more efficiently.

# Communicate

If you're jumping into open source work for the first time (Thanks and well
done!), you might see a Pull Request primarily as a formality. After all, the
people that own the project likely understand the code base better than you so
the feature you've added or bug you've fixed is self evident, right? Perhaps it
is. Even so, the Pull Request is your opportunity to communicate with the
project about your change.

Here are a few questions you might try to answer:

## What is the background that led to the change?

If this is a feature, spend a little time discussing what you needed to do but
were not able to with the current version. If it's a bug fix, give some
background on the conditions in which it reproduces. Providing some context
about how you arrived at the proposed solution will help bring reviewers up to
speed more quickly.

## What is the root cause?

For bug fixes, keep asking "Why?". Every "Why" you answer puts you one step
closer to understanding the root cause. The better you can describe the root
cause, the more confident you can be that your fix is suitable.

> Knowing the root cause doesn't necessarily mean that the proposed fix
> addresses that cause! Sometimes a workaround is appropriate but understanding
> the root cause means you know when you are applying a workaround.

## What alternatives did you try or consider?

For any reasonably complex system, there are several ways to solve a problem. If
you tried or considered alternatives, share what you tried and why you proposed
the solution you did.

* Did one have better performance than another?
* Where the performance trade-offs better (e.g. less memory usage vs faster
  runtime performance or the reverse)?
* How is the proposed solution more maintainable? Configurable?
* How does the user experience compare?

There are many more considerations but the important part is sharing what _you_
considered when implementing.

Also, don't shy away from sharing options that seem obvious but weren't chosen.
This will often head off "did you consider ..." questions since you've already
discussed why those options did not apply.

## What changed and how does it function?

Finally, discuss what you changed and how it implements the feature or fixes the
bug. If you normally start here, great! This is an important step and helps
reviewers understand the scope and behavior of the change before reviewing the
code. You might also find that the process of describing the change helps _you_
understand it better. If you're at all like me, you might even discover that
your proposed change doesn't quite do what you expect!

----

_What else have you found to be helpful in a pull request? Do you have a template for your project that has worked well? Reach out via a [Twitter](https://twitter.com/theryanjduffy), [email](ryan@tiqtech.com), or [GitHub Issue](https://github.com/ryanjduffy/blog/issues)._