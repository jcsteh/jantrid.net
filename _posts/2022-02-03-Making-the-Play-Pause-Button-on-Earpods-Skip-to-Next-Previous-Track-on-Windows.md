---
layout: post
title: 'Making the Play/Pause Button on Earpods Skip to Next/Previous Track on Windows'
tags:
- Earpods
- Windows
---

I recently got a new laptop: Dell XPS15 9510.
While this is a pretty nice machine overall, its audio drivers are an abomination.
Among other things, the Waves MaxxAudio software it ships with eventually leaks all of your system memory if you use audio constantly for hours, which is the case for screen reader users.
I eventually got fed up and disabled the Waves crap, but this makes it impossible for me to use the headset mic on my Earpods.

To work around that, I bought an [Apple USB-C to 3.5-mm Headphone Jack Adapter](https://www.apple.com/au/shop/product/MU7E2FE/A/usb-c-to-35-mm-headphone-jack-adapter).
As well as supporting the mic on the Earpods, this adapter also supports the volume and play/pause buttons!
However, play/pause only plays or pauses.
In contrast, on the iPhone, pressing it twice skips to the next track and pressing it thrice skips to the previous track.

I discovered that these buttons simply get sent as media key presses.
So, I wrote a little AutoHotkey script to intercept the play/pause button and translate double and triple presses into next track and previous track, respectively.

It also supports holding the button.
Single press and hold currently does nothing, but you could adjust this to do whatever you want.
On the iPhone, double/triple press and hold fast forwards/rewinds, respectively.
Support for triggering fast forward/rewind in Windows apps is sketchy - I couldn't get it to work anywhere - so these are currently mapped to shift+nextTrack and shift+previousTrack.
This way, you have the option of binding those keys in whatever app you're using.

You can [get the code](https://gist.github.com/jcsteh/75fd31cc3efc0016fc86f98f5ba02ca1) or [download a build](https://files.jantrid.net/playPauseMultiFunction.exe).
