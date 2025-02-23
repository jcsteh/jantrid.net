---
layout: post
title: 'Moving a Row in Google Sheets with the Keyboard'
tags:
- accessibility
- tips
---

Sometimes, it is necessary to move a row up or down in a Google Sheet.
For example, I might be maintaining a list of tasks ordered from highest to lowest priority and realise that a task later in the sheet is actually higher priority than earlier tasks.
[This Google support article](https://support.google.com/docs/answer/54813?hl=en&co=GENIE.Platform%3DDesktop#zippy=%2Cmove-rows-columns-or-cells) says you can do this by selecting the row and then choosing Move row up (or down) from the Edit menu.
However, when I selected the cells in the row, these Move row items still didn't appear in the Edit menu.

The solution is to press shift+space twice to select the row, after which the Move sub-menu will appear in the Edit menu, from which you can choose Row up or Row down.
Shift+space is the keyboard shortcut to select a row, but the first press only selects the cells in the row.
To select the row itself, which you must do in order to move it, you need to press it again.
Putting it all together, the entire keyboard sequence to move a row up in Firefox on Windows is shift+space, shift+space, alt+shift+e, m, k.
The keyboard shortcuts may be slightly different in other browsers or on Mac.

Moving columns is similar, except that you press control+space twice to select a column.

I couldn't find this documented or discussed anywhere, so I thought it worth posting here in the hopes that someone else might find it useful.
Thanks to [Josh Grams on Mastodon](https://icosahedron.website/@JoshGrams/113798362474991162) for figuring this out.
