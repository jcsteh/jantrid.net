---
layout: post
title: 'Cache the World: Turbo Charging Firefox Accessibility Performance and Maintainability'
tags:
- Firefox
- accessibility
---

The Firefox accessibility engine is responsible for providing assistive technologies like screen readers with the information they need to access web page content.
For the past couple of years, the Firefox accessibility team have been working on a major re-architecture of the accessibility engine to significantly improve its speed, reliability and maintainability.
We call this project "[Cache the World](https://wiki.mozilla.org/Accessibility/CacheTheWorld)".
In this post, I explain the reasons for such a massive undertaking and describe how the new architecture solves these issues.

## The need for speed
The biggest motivation for this project is to make Firefox faster when used with screen readers and other assistive technologies, particularly on Windows.
Let's start by taking a look at some numbers.
The table below provides the approximate time taken to perform various tasks with Firefox and the NVDA screen reader, both before and after this re-architecture.

| | Before (no cache) | After (with cache)
| - | - | -
Load [nsCSSFrameConstructor.cpp on Searchfox](https://searchfox.org/mozilla-central/source/layout/base/nsCSSFrameConstructor.cpp), which contains a table with over 12000 rows | 128 sec | 6 sec
Load the [WHATWG HTML spec](https://html.spec.whatwg.org/), a very large document | 175 sec | 15 sec
Open a Gmail message from the inbox | 200 ms | 100 ms
Close a Gmail message, returning to the inbox | 410 ms | 150 ms
Switch Slack channels | 620 ms | 330 ms

These times will differ widely depending on the computer used, whether the page has been loaded before, network speed, etc.
However, the relative comparison should give you some idea of the performance improvements provided by the new architecture.

So, why were things so slow in the first place?
To understand that, we must first take a little trip through browser history.
Note that I've glossed over some minor details below for the sake of brevity and simplicity.

## In the beginning
Once upon a time, browsers were much simpler than they are now.
The browser was a single operating system process.
Even if there were multiple tabs or documents with iframes, everything still happened within a single process.

This worked reasonably well for assistive technologies, which use the accessibility tree to get information about the user interface and web content.
Operating system accessibility APIs were already being used to expose and query accessibility trees in other applications.
Although these APIs had to be extended somewhat to expose the rich semantics and complex structure of web content, browsers used them in fundamentally the same way as any other application: a single accessibility tree exposed from a single process.

Assistive technologies sometimes need to make large numbers of queries to perform a task; e.g. locating the next heading on a page.
However, making many queries across processes can become very slow due to the overhead of context switching, copying and serialising data, etc.
To make this faster, some assistive technologies and operating system frameworks ran their own code inside the browser process, known as in-process code.
This way, large batches of queries could be executed very fast.
In particular, Windows screen readers query the entire accessibility tree and build their own representation of the document called a virtual buffer.

## Multiple processes
As the web grew rapidly in usage and complexity, so too did the risk of security exploits.
To improve performance, stability and security, browsers started to move different web pages into separate processes.

Internet Explorer 8 used different processes for different tabs, but a web page was still drawn in the same process in which the page was loaded.
The accessibility tree was also exposed from that same process and assistive technologies could still inject code into that process.
This meant that there was no change for assistive technologies, which could still get direct, fast access to the content.

To further improve security, Chrome took a stricter approach as a fundamental part of its foundational design.
Web content processes were sandboxed so that they had as little access as possible, delegating tasks requiring more privileges to other processes through tightly controlled communication channels.
This meant that assistive technologies could not access the web content process containing the accessibility tree, nor could they inject code into that process.
Several years later, Firefox adopted a similar design, resulting in similar problems for accessibility.
The discovery of the [Meltdown and Spectre attacks](https://meltdownattack.com) led both browsers to go even further and isolate iframes in their own processes, which made the situation even more complicated for accessibility.

## Chrome's solution
At first, [Chrome experimented with handling accessibility queries in the main UI process and relaying them to the appropriate web content process](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/accessibility/browser/how_a11y_works_2.md#a-dead_end_proxying-accessibility-requests).
Because accessibility API queries are synchronous, the entire UI and the web content process were blocked until each accessibility query completed and returned its result.
This made this unacceptably slow, especially for large batches of queries as described above.
This also caused some obscure stability and reliability issues.

Chrome abandoned that approach in favour of [caching the accessibility trees from all other processes in the main UI process](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/accessibility/browser/how_a11y_works_2.md#caching-the-full-accessibility-tree).
Rather than synchronous queries between processes, Chrome asynchronously pushes the accessibility trees from each web content process.
This does require some additional time and processor power when pages load and update, as well as using extra memory for the cache.
On the other hand, it means that assistive technologies have direct, in-process, fast access to the content as they did before in other browsers.

## Firefox's solution, take 1
Firefox was designed long before Chrome and long before the complex world which necessitated multiple, sandboxed processes.
This meant that re-architecting Firefox to use multiple processes was a massive undertaking which took years and a great deal of resources.
Great care had to be taken to ensure that Firefox remained reliable for the hundreds of millions of users who depended on it every day.

Firefox built a very minimal cache in the main process containing only the tree structure and the role (button, heading, etc.) of each node.
All other queries were relayed synchronously to the appropriate web content process.
On Linux and Mac, where large batches of queries are far less common and virtual buffers aren't used, this was acceptable for the most part.

On Windows, as Chrome discovered, this was completely unacceptable.
Not only was it unusably slow, it was very unstable due to the fact that COM (the Windows communication mechanism used by accessibility) allows re-entry; i.e. another call can be handled while an earlier call is still running.
The Firefox multi-process communication framework wasn't designed to handle re-entry.
Thus, another approach was required on Windows.

The accessibility team considered implementing a full cache of all accessibility trees in the main process.
However, Mozilla needed to ship multi-process Firefox as soon as possible.
It was believed that it would take too long to implement a full cache and get it right.
Getting anything wrong could result in the wrong information being communicated to assistive technologies, which could be potentially disastrous for users who already depended on Firefox.
There are also other downsides to a full cache as outlined earlier.

Instead, Firefox used some advanced (and somewhat obscure) features of COM to allow assistive technologies to communicate with the accessibility tree in content processes.
To mitigate the performance problems caused by large batches of queries, a partial cache was provided for each node.
Querying a node still required a synchronous, cross-process call, but instead of just returning one piece of information, the cache was populated with other commonly retrieved information for that single node.
This meant that some subsequent queries for that node were very fast, since they were answered from the cache.
All of this was done using a [COM lightweight client-side handler](https://learn.microsoft.com/en-us/windows/win32/com/the-lightweight-client-side-handler).
The entire cache for all nodes was invalidated whenever anything changed.
While naive, this reduced the risk of stale information.

The performance with assistive technologies took a massive step backwards when this was first released in Firefox 57.
Over time, we were able to improve this significantly by extending the COM handler cache.
Eventually, we reached a point where we could not improve the speed any further with the current architecture.
Because software other than assistive technology uses accessibility APIs (e.g. Windows touch, East Asian input methods and enterprise SSO tools), this was even impacting users without disabilities in some cases.
Furthermore, COM was never designed to handle the massive number of objects in many web accessibility trees, resulting in severe stability problems that are difficult or even impossible to fix.

The complexity of this architecture and the need for different implementations on different operating systems made the accessibility engine overly complex and difficult to maintain.
This is particularly important given the small size of our team.
When we revamped our Android and Mac implementations in 2019 and 2020, we had to implement more operating system specific tweaks to ensure decent performance, which took time and further complicated the code.
This wouldn't have been necessary with the full cache.
Of course, maintaining the caching code has its own cost.
However, this work can be more easily distributed across the entire team, rather than relying on the specific expertise of individual team members in particular operating systems.

## Enter Cache the World
Our existing architecture served us well for a few years.
However, as the problems began to mount, we decided to go back to the drawing board.
We concluded that the downsides of the full cache were far outweighed by the growing problems with our existing architecture and that careful design could help us mitigate those downsides.
Thus, the Cache the World project was born to re-architect the accessibility engine.

In the new architecture, similar to Chrome, Firefox asynchronously pushes the accessibility trees from each web content process to the main UI process.
When assistive technologies query the accessibility tree, all queries are answered from the cache without any calls between Firefox processes.
When a page updates, the content process asynchronously pushes a cache update to the main process.
The speed improvement has far exceeded our expectations, and unlike the old architecture, we still have a great deal of room to improve further, since we have complete control over how and when the cache is updated.
As for code maintenance, once this is fully released, we will be able to remove around 20,000 lines of code, with the majority of that being operating system specific.

## The journey to the world of caching
Aside from the code needed to manage the cache and update it for many different kinds of changes, this project required a few other major pieces of work worth mentioning.

First, Firefox's desktop UI is largely written using web technologies (HTML, CSS and JavaScript), but this must be rendered in the main process.
The cache isn't needed for this, but we wanted to share as much code as possible between the cached and non-cached implementations.
In particular, there is a layer of code to support the accessibility APIs specific to each operating system and we didn't want to maintain two completely separate versions of this.
So, we created a [unified accessibility tree](https://firefox-source-docs.mozilla.org/accessible/Architecture.html), with a [base `Accessible` class](https://searchfox.org/mozilla-central/source/accessible/basetypes/Accessible.h) providing an interface and functionality common to both implementations ([`LocalAccessible`](https://searchfox.org/mozilla-central/source/accessible/generic/LocalAccessible.h) and [`RemoteAccessible`](https://searchfox.org/mozilla-central/source/accessible/ipc/RemoteAccessibleBase.h)).
Other code, especially operating system specific code, then had to be updated accordingly to use this unified tree.

Second, the Windows specific accessibility code was previously entangled with the core accessibility code.
Rather than being a separate class hierarchy, Windows functionality was implemented in subclasses of what is now called `LocalAccessible`.
This made it impossible for the Windows code to support the separate cached implementation.
Fixing this involved separating the Windows implementation into a separate class hierarchy ([`MsaaAccessible`](https://searchfox.org/mozilla-central/source/accessible/windows/msaa/MsaaAccessible.h)).

Third, the code which provided access to text (words, lines, formatting, spelling errors, etc.) depended heavily on being able to query Firefox's layout engine directly.
It dealt with text containers rather than individual chunks of text, which was not ideal for efficient caching.
There were also a lot of bugs causing asymmetric and inconsistent results.
We replaced this with a completely new implementation based on text ranges called [`TextLeafRange`](https://searchfox.org/mozilla-central/source/accessible/base/TextLeafRange.h).
It still needs to use the layout engine to determine line boundaries, but it can do this for individual chunks of text and it provides symmetric, consistent results.
`TextLeafRange` is far better suited to the Mac text API and will make the [Windows UI Automation text pattern](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-about-text-and-textrange-patterns) much easier to implement when we get to that.
We also replaced the code for handling tables, which similarly depended heavily on the layout engine, with a new implementation called [`CachedTableAccessible`](https://searchfox.org/mozilla-central/source/accessible/base/CachedTableAccessible.h).

Fourth, our Android accessibility code required significant re-design.
On Android, unlike other operating systems, the Firefox browser engine lives in a separate thread from the Android UI.
Since accessibility queries arrive on the UI thread, we had to provide [thread-safe access to the accessibility cache on Android](https://firefox-source-docs.mozilla.org/accessible/GeckoViewThreadTopography.html).

Finally, screen coordinates and hit testing, which is used to figure out what node is at a particular point on the screen, were an interesting challenge.
Screen positioning on the modern web can be very complicated, involving scrolling, multiple layers, floating content, transforms (repositioning/translation, scaling, rotation, skew), etc.
We [cache the coordinates and size of each node relative to its parent](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/generic/LocalAccessible.cpp#3274) and separately cache [scroll positions](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/generic/LocalAccessible.cpp#3466) and [transforms](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/generic/LocalAccessible.cpp#3440).
This minimises cache updates when content is scrolled or moved.
Using this data, we then [calculate the absolute coordinates on demand when an assistive technology asks for them](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/ipc/RemoteAccessibleBase.cpp#545).
For hit testing, we [use the layout engine to determine which elements are visible on screen and sort them from the top layer to the bottom layer](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/generic/LocalAccessible.cpp#3173).
We cache this as a flat list of nodes called the viewport cache.
When an assistive technology asks for the node at a given point on the screen, we [walk that list, returning the first node which contains the given point](https://searchfox.org/mozilla-central/rev/57527d50ef5d3df412caa5d99536f0709399be6f/accessible/ipc/RemoteAccessibleBase.cpp#355).

## How is Firefox's cache different to Chrome's?
While Firefox's cache is similar to (and inspired by) Chrome's, there are some interesting differences.

First, to keep its cache up to date, [Chrome has a cache serialiser which is responsible for sending cache updates](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/accessibility/browser/how_a11y_works_2.md#render-process-accessibility-code-outside-of-blink).
It starts by notifying the serialiser that a node has changed.
The specific change is mostly irrelevant to the serialiser; it just re-serialises the entire node.
The serialiser keeps track of what nodes have already been sent.
When walking the tree, it sends any new nodes it encounters and ignores any nodes that were already sent and haven't been changed.

In contrast, Firefox uses its existing accessibility events and specific cache update requests to determine what changes to send.
When a node is added or removed, Firefox fires a show or hide event.
This event is used to send information about a subtree insertion or removal to the main process.
The web content process doesn't specifically track what nodes have been sent, but rather, it relies on the correctness of the show and hide events.
For other changes to nodes, Firefox uses existing events to trigger cache updates where possible.
Where it doesn't make sense to have an event, code has been added to trigger specific cache updates.
The cache updates only include the specific information that changed.
We've spent years refining the events we fire, and incorrect events tend to cause problems for assistive technologies and thus need to be fixed regardless, so we felt this was the best approach for Firefox.

Second, Chrome includes information about word boundaries in its cache.
In contrast, Firefox calculates word boundaries on demand, which saves memory and reduces cache update complexity.
We can do this because we have access to the code which calculates word boundaries in both our main process and our content processes, since our main process renders web content for our UI.

Third, hit testing is implemented differently.
I described how Firefox implements hit testing earlier.
Rather than maintaining a viewport cache, Chrome [first gets an approximate result using just the cached coordinates in the tree](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/accessibility/browser/how_a11y_works_3.md#hit-testing).
It then sends an asynchronous request to cache a more accurate result for the next query at a nearby point on the screen.
Our hope is that the viewport cache will make initial hit testing more accurate in Firefox, though this strategy may well need some refinement over time.

## So, when can I use this awesomeness?
I'm glad you asked!
The new architecture is [already enabled in Firefox Nightly](https://blog.mozilla.org/accessibility/significant-improvements-for-screen-readers-now-in-nightly-firefox/).
So far, we've received very positive feedback from users.
Assuming all continues to go well, we plan to enable this for Windows and Linux users in Firefox 110 beta in January 2023.
After that, we will roll this out in stages to Windows and Linux users in Firefox 111 or 112 release.
There is still a little work to do on Mac to fully benefit from the cache, particularly for text interaction, but we hope to release this for Mac soon after Windows.

Now, go forth and enjoy a cached world!
