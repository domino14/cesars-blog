+++
date = "2018-07-31T02:23:11+00:00"
draft = false
tags = ["devjournal"]
title = "Building an emulator"
+++
I have been coding in one form or another since I was around 8 years old, on my brother's calculator that had around 450 bytes of program memory, and I've been videogaming since I was 6 (my dad bringing an NES from the US back to my native Caracas was one of my happiest memories). But why not combine both those loves?

I've been wanting to make an emulator for ages but for some reason or another I've kept putting it off. I'm still always busy but I figure it's time to start a cool new project. My target will be emulating the SNES, probably my favorite console of all time, to the extent that it can play FFVI, Yoshi's Island, and Mario RPG as well as I can make them - the latter of those two use different coprocessor chips so those would be fun to emulate as well. As an electrical engineer by trade I do understand roughly what it would take to write a console emulator.

However, I hear the SNES is notoriously difficult to emulate. The bsnes creator has made it his [life's magnum opus](https://arstechnica.com/gaming/2011/08/accuracy-takes-power-one-mans-3ghz-quest-to-build-a-perfect-snes-emulator/) to write a perfect SNES emulator and it seems extremely involved. I don't necessarily need to emulate processor bugs in Speedy Gonzales, but something that can play games acceptably is probably fine for now; I just need not to let my perfectionism take over.

To start with, before anything, I plan to emulate the Game Boy, which should get me the practice needed to write something more involved. Apparently, it's rather easy to write a Game Boy emulator, and this was another console I played the hell out of (Link's Awakening FTW). I plan to use the Go language, which is one of my favorites, and whatever display libraries are used nowadays (OpenGL? even though apparently the Mac won't support it soon?). I'll post with more updates when I actually start.