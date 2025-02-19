---
layout: post
title: 'Pasting Numbers Without Punctuation or Spaces'
tags:
- clipboard
- tips
---

Everyone loves paying bills, right?
One of the best parts about paying bills is surely that they usually provide all the amounts, billing codes, reference numbers, etc. with punctuation and spaces; e.g. amount $1,234, reference code 123 456-789.
But banking and payment portals - sometimes even the payment portal for the organisation issuing the bill! - won't accept spaces and punctuation!
This means you can't simply copy and paste the numbers, which is really annoying.
Wouldn't it be awesom if there were some easy, automatic way to strip out everything except numbers and the decimal point before pasting?

I recently started using [Ditto](https://ditto-cp.sourceforge.io/), an absolutely superb clipboard manager for Windows.
It turns out you can do this with Ditto using a paste script.
I originally thought that paste scripts always ran, which I wouldn't want in this case, but it turns out that they actually appear as items in the Special Paste menu.

To set this up, go to Ditto -> Options -> General -> Advanced -> On Paste Scripts and add this script:

```
clip.AsciiTextReplaceRegex("[^\\d.]", "");
return false;
```

The name you specify for the script is what appears in the Special Paste menu.
You can even assign a shortcut key to that Special Paste item.
Then, you just have to copy the problematic number to the clipboard, open Ditto, go to the item you want to paste (which will be the top item if you just copied it) and press the shortcut key (or use the Special Paste menu).
Done!
Now you have 123456789 instead of 123 456-789, and your payment portal (and you) are much happier.
