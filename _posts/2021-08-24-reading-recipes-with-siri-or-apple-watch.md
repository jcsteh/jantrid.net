---
layout: post
title: 'Reading Recipes with Siri or Apple Watch'
tags:
- iPhone
- Apple Watch
- accessibility
---

I'm finally learning to cook some decent food, so I need to be able to read recipes.
For a while, I was reading them out of Simplenote on my iPhone.
However, I encountered several frustrations with this approach (and this applies to any notes or text app really):

- When you're not editing, Simplenote shows the note such that each line is an item for VoiceOver; i.e. you flick right to read the next line.
  However, if you bump the screen or perform the wrong gesture accidentally, you can easily lose your position.
  If the screen locks or you have to switch apps, you lose your position completely, since VoiceOver doesn't restore focus to the last focused item in apps.
- When editing, you can review the note line by line using the rotor.
  The advantage here is that the cursor doesn't get lost when you switch apps or the screen locks.
  However, lines can be smaller than is ideal due to the screen size, so one recipe instruction might get split across multiple lines.
  Also, moving the editing cursor with VoiceOver is notoriously buggy, often getting stuck, etc.
  Finally, again, if you bump the screen or perform the wrong gesture, you can lose your position (or worse, accidentally type text into the document).
- The screen lock problem could be solved by disabling auto lock, but that obviously has an impact on battery.
- Having to repeatedly take my phone out of my pocket to read the next instruction was impractical, especially given the risk of losing my spot in the recipe.
  Leaving it on a bench somewhere meant I had to keep walking back to wherever my phone was located, which was similarly tedious.
  This might seem simple enough, but when you're moving around a lot, using your hands for other things, getting your hands dirty, etc., it just isn't efficient.

I considered a couple of solutions:

1. I tried looking for an iOS app that could read recipes using Siri.
  If such an app exists, I couldn't find it.
  Any normal recipe app would likely have the same problems as above for VoiceOver users.
2. Google Home and Amazon Alexa can read recipes interactively using voice commands.
  I don't own either of those, but I was willing to consider the purchase.
  However, they can only read recipes from partner sites.
  This means you can't read recipes from other sources or recipes you've customised... and I tend to tweak recipes quite a bit for my own convenience.

So, I resigned myself to developing my own solution to read recipes with Siri.
This isn't specific to recipes.
It can be any line based text.
For example, it could be equally useful for other kinds of instructions where you need to be able to move step by step, but might have delays (maybe many minutes) between reading each step.

## How it Works
As explained above, I need to be able to edit and read customised recipes.
I find it much easier to edit long text on my laptop.
So, my solution takes the text from a simple text file stored on iCloud Drive.
This way, I can edit the text on my laptop, save it directly to iCloud Drive and have it reflected almost immediately in my reader solution without any extra effort.

Recipes usually have at least two sections (e.g. Ingredients and Method).
It's sometimes helpful to be able to jump between those.
The solution allows me to use a Markdown style heading (`# heading text`) to mark section headings.

The solution can be used while the phone is locked.
An added advantage of this is that it can even be triggered from Homepod with the responses read on the homepod, though I usually prefer to use my Airpods.

I can then use these Siri commands (i.e. after saying "Hey Siri"):

- Read next: Read the next line of text.
- Read previous: Read the previous line of text.
- Read repeat: Repeat the line of text that was last read.
- Read next section: Jump to the next section heading and read it.
- Read previous section: Jump to the previous section heading and read it.

In all cases, the solution keeps track of the last line that was read until you next use a command.
Even if I wait an hour, I'll still be exactly where I last left it.

Sometimes, I want to be able to quickly review many instructions; e.g. if I'm looking for multiple ingredients or reading ahead to see what's coming up.
In this case, I can use the Siri command "read browse" while the phone is unlocked.
This presents the instructions in a WebView so I can flick right and left between them with VoiceOver.
I can also use the headings rotor to move between headings.
When it opens, it focuses the line I last read.
Furthermore, if I double tap one of the lines, it sets that as the "bookmark"; i.e. the last line read.
For example, if I double tap the second instruction in the method section of a recipe and later say "Hey Siri, read next", Siri will read me the third instruction in the method section.

While in this browsing view, each line occupies almost the entire screen.
This might be useful if you're reading notes for a live talk you're giving and don't want to risk losing your spot if you bump the screen.

## Reading on Apple Watch
A few weeks ago, I bought an Apple Watch.
I began to wonder: could I somehow make use of the Apple Watch for something similar to "read browse"?

The Apple Watch has three nice advantages here:

1. It's on your wrist, so you don't have to worry about locating it, pickig it up, accidentally dropping it, etc.
2. Although it does go to sleep, you don't have to unlock it once it's on your wrist and unlocked.
3. It's much more water resistant than phones, so I'm less worried about sticking my grubby hands all over it.

Now, the solution works on Apple Watch too.
For technical reasons, it unfortunately can't focus the last line I read with Siri and doesn't support heading navigation.
When the Apple Watch wakes up after going to sleep, VoiceOver doesn't restore focus to the line I last read.
However, because the screen is so small and the scroll position is kept, I can just tap the screen to read the last line (or at least one very nearby), so this isn't a real problem on the watch.

Interestingly, I find I now use the watch to read recipes far more than Siri.
Before I got the watch, I'd been using the Siri solution with my phone for a few months and was reasonably happy with it.
However, speaking Siri commands can be slow if you're reading several instructions in quick succession.
Also, Siri would sometimes misunderstand my commands; e.g. trying to read text messages instead of "read next".
Also, I found I wanted to read ahead more often with some recipes and having to find my phone, pick it up, unlock it, etc. to use "read browse" was slightly annoying.

That said, I suspect I'll still use Siri in some cases.
It's useful being able to interactively read instructions in multiple ways depending on what I need at the time.

## Implementation
The current solution is implemented using iOS Shortcuts and [Scriptable](https://scriptable.app/).
Scriptable is an absolutely fantastic iOS app that allows you to "automate iOS using JavaScript".
It exposes many iOS APIs in JavaScript and allows integration with Siri shortcuts, among many other features.

[Here's the code for the Scriptable script](https://gist.github.com/jcsteh/0d9238bab60b4accc61ffc1642c0e468).

Unfortunately, getting this set up is pretty tedious because you have to manually create a bunch of Siri shortcuts.

1. Import the script into Scriptable.
  The easiest way to do this is to copy the file into the Scriptable folder on iCloud Drive.
2. In the Shortcuts app, create a shortcut for "Read next":
  - Add the Scriptable "Run Script" action and choose the SiriInteractiveReader script.
  - Tap the "Show More" button for that action.
  - Under "Texts", add a new item and enter the text: nextLine
  - Ensure "Run In App" and "Show when Run" are both off.
  - Add the "Show Result" action.
  - Name the shortcut "Read next".
3. Duplicate this shortcut for the rest of the Siri reading commands.
  Aside from the name of the shortcut, the difference in each shortcut will be the text entered under "Texts" in the "Run Script" action:
  - For Read previous: previousLine
  - For Read repeat: repeatLine
  - For Read next section: nextSection
  - For Read previous section: previousSection
4. For the "Read browse" shortcut, the text under "Texts" should be "browse".
  "Run In App" must be on.
  The "Show Result" action should be removed (so there's only the "Run Script" action).
5. If you have an Apple Watch, you can add a shortcut to support this.
  - Again, you need the "Run Script" action with the script set to SiriInteractiveReader.
  - Under Texts, enter the text: list
  - Add the "Get Dictionary Value" action.
  - Set the key for that action to: list
  - Add the "Choose Item from List" action.
  - Name the shortcut "Read watch" and ensure "Show on Apple Watch" is on.
  - Note that you should run this action from the Shortcuts app on the watch, not from Siri.

The text you want to read should be placed in a file called `SiriInteractiveReader.txt` in the `Scriptable` folder on iCloud Drive.

## Learnings
I learned a great deal throughout the process of implementing this.
Here are some learnings that might be of interest to others working with iOS Shortcuts and Scriptable.

1. If you want to have Siri read a response without also saying "That's done" or similar, you need to turn off "Show when Run", return the text as output from your Scriptable script and use the "Show Result" action in Shortcuts.
  The intuitive way to speak text using Siri is to use Scriptable's [Speech.speak function](https://docs.scriptable.app/speech/#speak).
  If you do this, Siri seems to want to speak "That's done" or similar.
  In contrast, this doesn't happen when you use "Show Result".
  The added advantage is that the shortcut will display the text on screen if run outside of Siri.
2. If you have Scriptable present a WebView with "Run In App" turned off, you won't be able to activate anything in the WebView.
3. The `alert` and `window.close` functions don't work in Scriptable WebViews.
4. Scriptable's [WebView.evaluateJavaScript function](https://docs.scriptable.app/webview/#-evaluatejavascript) doesn't work correctly while the WebView is being presented.
  I was hoping to use this to handle moving the reading bookmark when the user taps a line.
  Instead, I used a [`scriptable://` URL](https://docs.scriptable.app/urlscheme/) to open the script with a specific parameter, which also dismisses the WebView.
5. There is no Scriptable app for Apple Watch (yet).
  However, as long as you don't try to present any UI whatsoever from within Scriptable, you can still make use of Scriptable in shortcuts run on the watch.
  The script will run on the phone.
  You can still present UI, but you have to do it with Shortcuts actions, which are able to run on the watch.
  You can get creative here to present UI based on output from a Scriptable script, as I do using the "Choose Item from List" in the watch shortcut above.

## The Future
Ideally, it'd be good to develop this into an app so it's not so tedious (probably impossible for many users) to install.
I considered this, but I'm one of these strange developers that still uses a text editor and prefers to design GUI using markup languages or code.
The prospect of learning and using Xcode and having to use a GUI builder is not something I'm at all motivated to do in my spare time.
I've read you can design iOS GUI in code to some extent, but it looks super painful.
