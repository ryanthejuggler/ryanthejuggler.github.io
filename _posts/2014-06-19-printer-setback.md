---
layout: post
title: "Printer setback"
date: 2014-06-19 13:05:24
categories: 3D-printing
---

I had a bit of time last night to work on my printer. Unfortunately, the universe had other plans, and I only managed to get a bit of coding done.

First, some context. We moved on June 1 to a tiny apartment in [Beacon Hill](http://en.wikipedia.org/wiki/Beacon_Hill,_Boston), and created the illusion of having unpacked due to a very deep closet. Unfortunately, what we should have done instead is *actually* unpacked; periodically I find myself shuffling through boxes and boxes just to find one item.

Last night was just such an occasion. I needed the USB-B cable that goes to the [RAMBo](http://reprap.org/wiki/Rambo) board on my printer, and I knew it was in the closet somewhere. However, the closet was so stacked with stuff that it was pretty difficult to move the boxes around. What was initially a quick search for a cable turned into about 90 minutes of cleaning and reorganizing. I took all the boxes and spread them out on the floor across the entire apartment before stacking them back up in a much more organized fashion. (Upside of this is that now we have a bit more room in the closet, and it's easier to find things.)

Cable in hand, I connected my printer to my laptop. I then proceeded to spend the next hour fighting with the Play Framework; the WebSockets I was creating would open, respond once, and then become unresponsive. The cause was that I had a match statement from an earlier test that was failing, but it took me an hour of poking around to figure this out. Finally, I used the [echo test page at websocket.org](http://www.websocket.org/echo.html) to send commands to my printer. I connected to it and sent the "home" command.

The machine started moving very erratically and noisily. I slammed down the power switch and inspected it closely. Turns out that because I left the motors plugged in during the move, the motor clips had gotten bent out of shape and no longer made good contact. I'm now going to have to resolder those clips before my printer moves smoothly again.

At this point I just gave up and went to bed. Maybe I'll try again tonight.
