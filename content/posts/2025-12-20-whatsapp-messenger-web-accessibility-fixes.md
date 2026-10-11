---
title: 'WhatsApp Web and Messenger Accessibility Fixes'
url: /2025/12/20/whatsapp-messenger-web-accessibility-fixes/
tags:
- accessibility
---

## WhatsApp
If you use WhatsApp on Windows, you are likely aware by now that WhatsApp have discontinued the UWP app and replaced it with an app that effectively just wraps the WhatsApp web app.
Unfortunately, while the accessibility of the UWP app was quite decent, the web app has some major accessibility problems which have not yet been resolved.
Fortunately, one of the nice things about the web is that it's possible to tweak accessibility a great deal with scripting.
This can only be done if you run the web app in a web browser rather than the desktop app, but since the desktop app basically just wraps the web app anyway, there's no real disadvantage to doing this.

If you want better accessibility for WhatsApp on Windows and you're willing to run WhatsApp web in Firefox, you might wish to try my WhatsApp Web Accessibility Fixes Greasemonkey script.
You might be able to get this to work in Chromium browsers too using a different extension, but I haven't tried this and it's not something I can help with.

Here's how to set it up:

1. In Firefox, install [the Greasemonkey add-on](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/).
2. Install my [WhatsApp Web Accessibility Fixes Greasemonkey script](https://github.com/jcsteh/axSGrease/raw/master/WhatsAppA11yFixes.user.js).
3. Go to [web.whatsapp.com](https://web.whatsapp.com/).
4. If you wish, you can even [add the site to the taskbar](https://support.mozilla.org/en-US/kb/web-apps-firefox-windows) so it behaves more like a standalone app using the "Add tab to taskbar" button on the toolbar.

If you use NVDA, I suggest you switch to focus mode with NVDA+space and stay there unless you need to review something in browse mode.
Then:

1. Pressing alt+1 focuses the first chat in the Chats list.
2. Pressing alt+2 focuses the last message in the active chat.
3. Pressing the context menu key opens the context menu for the current chat or message.
4. With NVDA, the reaction picker no longer switches to browse mode and starts reading all the reactions.
5. The script also prevents WhatsApp from forcing focus back into the message list when you switch tabs, regardless of what was previously focused.
6. Note also that pressing any character while focused in the message list will focus the message composer, but that's not part of my script; WhatsApp Web already does that.
7. The [web app also provides other keyboard shortcuts](https://faq.whatsapp.com/6204576529560565/?cms_platform=web) that might be useful.

## Facebook Messenger
Messenger no longer has a Windows desktop app at all.
I've also written a [Facebook Messenger Accessibility Fixes Greasemonkey script](https://github.com/jcsteh/axSGrease/raw/master/MessengerA11yFixes.user.js) with similar functionality.
You can install it similar to the instructions for WhatsApp above.
Once installed, go to [messenger.com](https://messenger.com/).

Again, if you use NVDA, I suggest you switch to focus mode with NVDA+space and stay there unless you need to review something in browse mode.
Then:

1. Pressing alt+1 focuses the first chat in the Chats list.
2. Pressing alt+2 focuses the last message in the active chat.
3. Pressing alt+3 focuses the message composer.
4. Pressing enter (not the context menu key) on a message opens the actions for that message, though Messenger already supported this.
