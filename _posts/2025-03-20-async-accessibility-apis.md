---
layout: post
title: 'My Thoughts on Asynchronous Accessibility APIs'
tags:
- accessibility
- web
---

Accessibility API queries are generally synchronous: each query blocks both the client and the server until the query completes and returns its result.
This causes [significant challenges for modern, multi-process web browsers, requiring them to cache the accessibility trees from all other processes in the main UI process](/2022/12/22/Cache-the-World/).
This raises the question: why not make accessibility APIs asynchronous?
As usual, there is a theoretical/principle answer and a pragmatic answer.

Let's start with theory.
For a browser, synchronous accessibility APIs are a real problem.
So much of the complexity in multi-process browser accessibility architecture comes back to the need to support synchronous APIs.
If they were async, we might not need to maintain a cache in the main process at all, which would save us a lot of pain, complexity and performance problems.
From that perspective, async APIs would be great.
Async APIs would likely also help assistive technology products to avoid hangs due to queries taking a long time or apps which stop responding, which is a real source of pain for users.

But here's where pragmatism comes in.
Between 2017 and early 2023, Firefox didn't have a full cache in the parent process.
In Windows, [we used a lot of obscure COM magic to facilitate a partial cache](/2022/12/22/Cache-the-World/#firefoxs-solution-take-1).
It worked better than I ever thought it would, but it still suffered from serious performance and stability problems.
Ultimately, we took the same route as Chromium and [implemented a full cache](/2022/12/22/Cache-the-World/#enter-cache-the-world).
But before that, I considered a lot of other options, including looking into the possibility of creating an entirely new async API.

In the end, I gave up on all other options, as much as it pained me.
I came to understand that it would just require too much change across the entire industry to pull this off:

1. Getting this to work in Windows alone would require significant new code in all browsers, all assistive technology products (including commercial ones) and likely even Windows itself.
2. It would fundamentally change the way just about everything works in AT, since a lot of AT code is rather centred around the idea of sync queries.
3. Worse, all of the software would still need to support the old APIs while the new ones developed and became adopted, so we'd all be trying to support sync and async paradigms simultaneously, as if supporting async by itself wasn't hard enough.
4. Commercial AT users often don't update for years because the software is so expensive, so that would mean a further massive delay to reasonable adoption.
5. For providers other than browsers, there isn't really an incentive to adopt this.
  For most providers, sync APIs work just fine.
  Browsers are somewhat unique in their multi-process complexity.
6. Even if we got it working in Windows, we'd still have to work hard to get other operating system vendors (Apple, Google, Linux folks, etc.) on board, and even if they agreed, they would have to get around to doing that work.
7. We couldn't get rid of any of the complex architecture in browsers until everyone had made the switch.
8. Even assuming all of that happened, it could take many years.
9. Meanwhile, as far as I'm aware, accessibility everywhere is significantly under-resourced as it is and we have so, so many problems to solve that impact real users every day.
  A massive effort like this would make that other work even harder to fit in, even if it made some of it slightly easier.

None of this is to suggest that I am fundamentally against the idea of async APIs.
I'd potentially even be happy to be involved in such a project and perhaps we can try to make progress in small ways.
But at this point, I'm skeptical about the return on investment.
