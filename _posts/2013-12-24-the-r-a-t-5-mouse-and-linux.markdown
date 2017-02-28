---
layout: post
title: The R.A.T. 5 mouse and Linux
date: '2013-12-24 16:14:48'
tags:
- mad-catz
- mouse
- ubuntu
- linux
---

I recently purchased a **R.A.T. 5** since my previous mouse had failed to help me when I was at a shortage of ammo and within an inch of my life playing [_Metro: Last Light_](http://enterthemetro.com/), specifically, the left button would fail to make contact.

![rat5](/content/images/2013/Dec/SAITEKR_A_T5.jpg)

Put aside the reason why I threw my wallet at a relatively expensive mouse, let's keep it at the abundance of buttons, but most important to me: it just feels **right** in the palm of my hand.

When plugging it in my laptop runnning Ubuntu (13.10), I noticed that it was acting very strangely.

The focus of the mouse pointer wasn't always the same as the focus on screen which was very annoying at times.

In that situation I had to press the upper left button until the focus was right.

I do believe that that would make sense on inferior Operating Systems with only one screen to rule them all, but to me that was a big annoyance.

Here's a snippet of code I added to **/etc/X11/xorg.conf** that fixed it for me after loging out and back again:

	Section "InputClass"
		Identifier "Mad Catz R.A.T. 5"
		MatchProduct "Mad Catz Mad Catz R.A.T.5 Mouse"
		MatchDevicePath "/dev/input/event*"
		Option "Buttons" "21"
		Option "ButtonMapping" "1 2 3 4 5 0 0 11 10 7 6 8 0 0 0 0"
		Option "ZAxisMapping" "4 5 11 10"
		Option "AutoReleaseButtons" "13 14 15"
	EndSection


After that my mouse has been behaving perfectly.