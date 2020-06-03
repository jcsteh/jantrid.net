---
layout: post
title: 'Tab Keywords in Firefox'
tags:
- Firefox
- keyboard
---

## Introduction
An awesome feature in Firefox that has existed forever is the ability to [assign keywords to bookmarks](https://support.mozilla.org/en-US/kb/bookmarks-firefox#w_how-to-use-keywords-with-bookmarks).
For example, I could assign the word "bank" to take me directly to the login page for my bank.
Then, all I have to do is type "bank" into the address bar and press enter, and I'm there.

Another awesome feature in Firefox is the ability to [use the address bar to switch to open tabs](https://support.mozilla.org/en-US/kb/address-bar-autocomplete-firefox#w_changing-results-on-the-fly).
For example, if I want to switch to my Twitter tab, I can type "% twitter" into the address bar, then press down arrow and enter, and I'm there.

Inspired by these two features, I started to wonder: what if you could have tab keywords to quickly switch to tabs you use a lot?
If you only have 8 tabs you use a lot, you can switch to the first 8 tabs with control+1 through control+8.
If you have more than that, you can search them with the address bar, but that gets messy if you have multiple pages with similar titles or a page title doesn't contain keywords that are quick to search.
For example, if you have both Facebook Messenger and Twitter Direct Messages open, you can't just type "% mes" because that will match both.
If you're on bug triage duty and have a bug list open, the list might not have a uesful title.
Wouldn't it be nice to just type "tm" to switch to Twitter Direct Messages or "fm" to switch to Facebook Messenger?

## Implementation
Now you can!

Trying to integrate this into the Firefox address bar seemed pretty weird.
Among other things, it wasn't clear what a good user experience would be for setting a keyword for a tab.
So, I decided to do this in an add-on.
Rather than writing my own from scratch, I found [Fast Tab Switcher](https://addons.mozilla.org/en-US/firefox/addon/fast-tab-switcher/) and [contributed the feature to that](https://github.com/tapapax/firefox-fts/pull/44).
Version 2.7.0 of Fast Tab Switcher has now been released which includes this feature.

## How it Works
First, install [Fast Tab Switcher](https://addons.mozilla.org/en-US/firefox/addon/fast-tab-switcher/) if you haven't already.

Then, to assign a keyword to a tab:

1. Switch to the tab.
2. Press control+space to open Fast Tab Switcher.
3. Type the `=` (equals) character into the text box.
4. Type the keyword to assign and press enter.

For example, to assign the keyword "fm", you would press control+space, type "=fm" and press enter.

To switch to a tab using its keyword, press control+space, type the keyword and press enter.
Note that the keyword must be an exact match.

Keywords stay assigned to tabs even if you close firefox, as long as you have Firefox set to restore tabs.

Enjoy super fast tab switching!
