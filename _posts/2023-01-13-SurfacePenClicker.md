---
layout: post
title:  "Using your Surface Pen as a clicker outside of PowerPoint"
---

# TL;DR

1. Download [Windows PowerToys](https://github.com/microsoft/PowerToys)
2. Goto Keyboard-Manager and create a new *Key Combination*
3. Press the pen short once and set it to the `left arrow` key.
4. Press the pen for a long period and set it to `right arrow` key.
5. Set your preferred PDF viewer as the *destination app* so your default keybindings will still work outside of presenting.
6. Profit.

> I use [impressive](https://impressive.sourceforge.net/) to present my PDF slides.

# Story

I already have a Surface Pen and Laptop that I don't use anymore
because it is glued shut and I would need more RAM.
Anyways, I am now creating slides with `beamer` and need to present a PDF file.
I already knew you could use the pen to switch slides in PowerPoint,
so there should be a way to override the default shortcuts of the pen when using another app.
I was about to google for a library to be able to change the keybindings.
Then I remembered PowerToys and the ability to overwrite keybindings and to my surprise it worked.

## Don't make the mistake of only overwriting one key.
Apparently when clicking on your pen, it is a key combination (`Win (Left) + F19`).
If you only overwrite `F19` to `left arrow` then your application windows will get moved.

