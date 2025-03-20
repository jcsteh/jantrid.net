---
layout: post
title: 'Direct UIA Access to Web Content Processes'
tags:
- accessibility
- web
- Windows
---

Before settling on the project to [implement a full accessibility cache in Firefox](/2022/12/22/Cache-the-World/#enter-cache-the-world), I investigated several other alternatives.
In my last post, I discussed [asynchronous accessibility APIs](/2025/03/20/async-accessibility-apis/).
Another alternative I considered is switching to UI Automation and having the UIA tree accessed directly in web content processes, rather than communicating via the main UI process.
While this is not possible with IAccessible2 because of browser sandboxes, it could theoretically be possible with UIA because it has an intermediary Windows component called UIAutomationCore which sits between the client and the server.
As an operating system component, there are less concerns with regard to allowing it in the browser sandbox.
Unfortunately, this proved to be infeasible then and I think it is still infeasible now:

1. First and foremost, UIA would need to be enhanced to support this.
2. UIA calls would need to arrive on a background worker thread.
  Providers can specify ProviderOptions_UseComThreading to ensure that calls come in on the correct COM thread, which both Firefox and Chromium do in the main process so that calls hit the main (STA) thread.
  However, Firefox (and Chromium) content processes are sandboxed and do not have a UI.
  In Win32 terminology, they don't have an HWND, and in fact the sandbox blocks Win32k calls altogether these days.
  That means that there is no COM STA in the main thread, so all UIA calls have to be handled from the background worker thread.
3. That is problematic for both browsers because accessibility needs to be called on the main thread.
  Proxying individual calls from the UIA thread to the main thread isn't workable because UIA might be handling a bulk request such as a tree search or caching request which makes many calls to the provider in quick succession.
  To fix this, UIA would need to introduce some mechanism to allow a provider to marshal and run the entire batch in a different context and then marshal the result back to UIACore.
  Such a mechanism does exist internally to some extent in order to support ProviderOptions_UseComThreading, but this would need to be extended further.
4. While it's reasonable for users to see tabs as entirely separate, users perceive iframes as just another part of the content.
  The iframe isn't a separate document as far as the user is concerned, even though it is very separate in the web stack.
  With site isolation (where iframes are rendered in separate processes), that is extremely problematic for UIA because each iframe is sandboxed in its own process.
  While a UIA client can happily unify the tree structure across processes, there are many things it cannot handle across processes.
  For example, the text pattern has no mechanism for walking across process boundaries, and remote ops is also explicitly constrained to a single process.
  That means that assistive technology products would have to treat iframes as entirely separate entities, which gets really messy (if not impossible) for things like quick navigation or text searching.
5. Assuming we could solve all of the above, there's still the problem that [UIA isn't really suitable for the web](/2025/03/19/why-uia-insufficient-web/).
  That would all need to be fixed in order for AT to seriously consider switching to UIA, which it would need to do in order for browsers to even consider dropping IA2.
6. Even if we could get NVDA to do this - and it does already have some usable support for UIA on the web - we would need to convince other AT vendors to do this and wait for it to be done.
  I think the chances of convincing other vendors are extremely low.
7. Even if we wanted to switch to this approach now, we still couldn't drop the full accessibility cache unless we could get other platforms to adopt an equivalent mechanism.
  Mac and ATK both have mechanisms for cross-process accessibility embedding, but I believe they (Mac at least) suffer from the same problems regarding isolated iframes; see point 4 above.
