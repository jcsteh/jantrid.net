---
layout: post
title: 'Why UI Automation is Insufficient as an Accessibility API for the Web'
tags:
- accessibility
- web
- Windows
---

[UI Automation](https://learn.microsoft.com/en-us/windows/win32/winauto/entry-uiauto-win32) (UIA) is Microsoft's recommended accessibility framework for Windows, replacing the earlier [Microsoft Active Accessibility](https://learn.microsoft.com/en-us/windows/win32/winauto/microsoft-active-accessibility) (MSAA) framework.
Despite this, to access web content, screen readers such as NVDA and JAWS continue to use [IAccessible2](https://github.com/LinuxA11y/IAccessible2), an open source API based on MSAA.
This is not just because of the significant effort involved in switching to a different API, although that is certainly a factor.
More importantly, it is because UIA is currently insufficient as an accessibility API for the web.
Here are some of the reasons:

1. UIA relies on LocalizedControlType to distinguish many controls, rather than adding more proper semantic ControlTypes.
  This means that aside from using a different message to report the control, clients can't reliably identify the types of these controls in order to choose different behaviour for them.
2. This also extends to landmark types that UIA doesn't support.
  For these, LandmarkType is specified as custom and the client can only use LocalizedLandmarkType to distinguish them.
3. This conflation of semantics continues to properties like aria-errormessage.
  In UIA, aria-errormessage gets mapped to ControllerFor, with no indication that this is an error message.
4. The LabeledBy property only allows the provider to expose a single static text target.
  This is problematic on the web, where an element can be labelled by multiple targets which may not even contain static text or may contain multiple static text nodes.
5. There is a single live region changed event.
  This doesn't provide sufficient granularity to determine what text actually changed in the live region, which is problematic for non-atomic live regions.
  While the eventual goal is to replace live regions with ariaNotification, live regions are still going to be around for a while.
6. Ideally, ARIA would map cleanly to each OS API, appearing just like a native application, with the API being enhanced as new concepts are needed.
  Instead, UIA has properties like AriaProperties and AriaRole, which smush all of the ARIA stuff that doesn't fit UIA properly into a separate world.
  I understand the need for pragmatism - this is probably never goingto be 100% clean - but some of this feels like a bolted on solution, treating the web as a second class citizen, rather than trying to harmoniously integrate it into the rest of the ecosystem.
  As an example, being able to determine whether a text box is multi line or single line feels like something that should be possible everywhere, but UIA instead shoves it into AriaProperties.
7. Historically, Microsoft has been unwilling (or at least very reluctant) to consider changes to UIA to improve this situation.
  For example, for many years, UIA didn't provide any way to identify a dialog or to retrieve group position information, despite me requesting this multiple times right back to 2009.
  These two deficiencies were eventually addressed years later with the addition of the IsDialog, PositionInSet and SetSize properties.
  Even so, Microsoft is going to need to be a lot more open (or at least faster to respond) to improvements like these if UIA is to become fully sufficient for the web, especially given the rapid pace of web technology.

All of this is rather unfortunate because UIA has a relatively clean design and has the potential to provide significant advantages for accessibility in performance, security, etc.
