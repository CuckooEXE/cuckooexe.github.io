---
layout: post
title: Finding the PerfectPitch
description: Learning how to write Flipper Zero apps to unsuccessfully break into my apartment.
image: https://raw.githubusercontent.com/CuckooEXE/DTMF-Generator/main/screenshot.png
excerpt_separator: <!--more-->
categories:
- Software Development
author:
- Axel Persinger
---

My apartment has a gated courtyard that you have to enter before being able to reach the front door. On the entrance to the gate is an RFID reader for a [DK1-3 Key Fob](https://www.farpointedata.com/downloads/datasheets/DK1_TDS.pdf) that was issued to us when we moved in, and a callbox. The call box has an LCD display, full keypad, and a few "meta" keys used to navigate through the system. When you wake up the callbox, a list of resident's names appear (which I'm not a huge fan of...), with the option to call. Once you call a resident, it gets forwarded to their phone with two-way audio, so you can talk to each other, but more importantly, the resident can press `0` on their phone to open up the gate.  

I was chatting with a friend when the thought came up - maybe I can play the same `0` audio tone from the callbox and trick it into opening up. An interesting idea, let's try it!

<!--more-->

## First Attempt

I stood outside, called my partner's phone via the callbox, and navigated to a DMTF tone generator site on my phone. I held down the `0` key, but no luck, the gate didn't open. To confirm the tone was correct, I played the `0` tone from *my* phone into my partner's phone's microphone. The gate opened! So the tone was correct, but the callbox wasn't picking it up. It was at this point that I realized, this probably wouldn't work. If I were the callbox's manufacturer, I would make sure to only process the signal coming in from the resident's phone, and *not* from the callbox. But I figured, maybe the speaker wasn't loud enough on my phone (it was pretty windy), so I need another source of sound.

## Flipper Zero

Like any good hacker, I buy fancy toys and never get around to actually using them (see also: Raspberry Pi). I figured this was as good as time as any to try and use the Flipper Zero, as I am pretty sure it has a tiny speaker built into it. One of my 2024 resolutions is to learn more technologies (stay tuned as I learn Bluetooth and ZigBee), and building a cool app for the Flipper Zero seemed like a good way to learn more about it.

Programming for the Flipper Zero took about an hour for me to wrap my head around. They have their own API and style for handling state, key presses, etc. It's not *too* complicated, but it just takes a minute to get used to. Anyway, I whipped up a super simple application that displays the valid DMTF Tones, let's you play one, and increase/decrease the volume. And by "a super simple application" I mean I made the UI/canvas first, which took probably four hours, and then the actual code took about 30 minutes. I'm not a UI/UX designer, so I'm pretty proud of how it turned out.

Right as I was implementing the actual tone generation code, I encountered some issues with the speaker on my Flipper Zero, so I asked a question on the official Discord. Someone said that there was already a DTMF tone generator application for the Flipper Zero. 🤦‍♂️ Why didn't I check first? I downloaded it, and it worked great. Now it's time to test it out on the callbox.

### DTMF Generator Application

I still decided to go ahead and upload my [DTMF Application to GitHub](https://github.com/CuckooEXE/DTMF-Generator); writing it was a fun learning experience, and that's my goal with all of these projects.

## Second Attempt

I went back outside to my apartment's callbox and called my own phone this time (my partner was busy, *rude!*), and stuffed it in my pocket so it wouldn't hear the DTMF tone from my Flipper Zero. Opened the DTMF Dolphin application, navigated to the `0` tone, and pressed play... Nothing. I tried it one more time but still no dice. My two observations are: my Flipper Zero's speaker is *incredibly* quiet, so it might just not be picking it up; and the tone sounds *slightly* different coming from my Flipper Zero than it does when I press the `0` button on my phone.

## Conclusion

At this point, I'm pretty confident that what I'm describing won't actually work. I would hope that the developers of the callbox ensured that the DTMF tones are only valid when coming from the resident's phone :). I'm no expert in DTMF tones, callboxes, etc. so if you have any ideas, please let me know! I'm always down to learn something new.

Make sure you do your research first for prior art, no reason to reinvent the wheel, **but** there's also nothing wrong (in fact, I'd encourage!) with reinventing the wheel for the sake of learning. 
