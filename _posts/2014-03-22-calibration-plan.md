---
layout: post
title: "Calibration plan"
date: 2014-03-22 15:08:00
categories: 3D-printing
---
Rather than building a touch probe (which is potentially prone to crashes), I'll be calibrating my printer using optical techniques I picked up while working on my undergrad thesis.

## Algorithm ##

The overall plan is as follows:

1. Capture several images (10 is a good number) of the calibration standard from different angles.
1. Locate the points of interest (POI) of the standard.
1. Use several POI sets to calibrate the camera. This gives us its intrinsic parameters such as focal length and distortion.
1. Turn the lasers on and move the print head along the $Z$-axis.
1. For each location along the $Z$-axis, store the $Z$ value as well as the $(x,y)$ locations of the laser spots on the standard.
1. Find a best-fit line equation for each beam given its set of $(x,y,z)$ coordinates found in the previous step.
1. You may now move the print head to any location and measure its location by using the intersection of the beams with the standard and solving for the print head location.
1. Iterate through possible sets of calibration parameters and find the one with the least amount of error.

I could probably do this analytically, but honestly, this method will get the results I want. I can always improve upon it later as well. But first, I have to fix the...

## <strike>Problem with the existing Marlin firmware</strike> ##

<strike>Yes, there's a clear bug in it!</strike> **UPDATE (May 7, 2014):** there's no problem at all! I've indented my original comments and addressed the flaw in my argument below.

> According to [JC Rocholl's blog](http://deltabot.tumblr.com/post/27300149759/hack-a-day-has-a-very-good-summary-of-rostock) (via [Hack A Day](http://hackaday.com/2012/07/13/3d-printing-with-a-delta-robot-that-seems-to-simplify-the-concept/)), the deltabot positioning algorithm proceeds as follows (I've formatted the equations myself):

> > First define three vectors from the origin to each support/drive pillar: $\vec A$, $\vec B$, $\vec C$
> > 
> > Define a vector from the origin to the desired location of the print head: $\vec W$
> > 
> > Define the length of an arm: $r$
> >
> > Define the height of each carriage on its pillar: $h\_a$, $h\_b$, $h\_c$
> > 
> > Then the horizontal distance from each pillar to the print head is:
> > <div> $$ \begin {aligned}
a & = \lVert \vec W - \vec A \rVert \\\\
b & = \lVert \vec W - \vec B \rVert \\\\
c & = \lVert \vec W - \vec C \rVert 
\end {aligned} $$ </div>
> >
> > Then, knowing the length of the arm, each height is defined as:
> > <div> $$\begin{aligned}
h\_a & = z + \sqrt{r^2 – a^2} \\\\
h\_b & = z + \sqrt{r^2 – b^2} \\\\
h\_c & = z + \sqrt{r^2 – c^2}
\end{aligned} $$ </div>
> >
> > This is actually pretty easy math!

> Easy indeed, Mr. Annirak, but you're forgetting something. This algorithm works perfectly as intended if you assume that all three arms meet at a single point. In reality, they do not meet at all. For instance, on my machine, the arms meet the center platform at a distance of 33 millimeters from the central axis. The above set of equations fails to take this into account. The least-invasive way to fix this is to define $\vec W\_a$, $\vec W\_b$, and $\vec W\_c$ as the desired position of each end of the print arm. Then simply replace $\vec W$ everywhere in the "horizontal distance" equations and you should be good to go.

> In practice, it was pretty simple to find where the fix needs to go. Despite the fact that it holds oodles of functions (3250 lines of code! So much for short files), [Marlin\_main.cpp](https://github.com/ErikZalm/Marlin/blob/51c6bd6b72aec22dc16dbbd6a9f37b81850d6fad/Marlin/Marlin_main.cpp) only refers to the `DELTA` macro a handful of times, and so it wasn't too hard to find the [`calculate_delta` function](https://github.com/ErikZalm/Marlin/blob/51c6bd6b72aec22dc16dbbd6a9f37b81850d6fad/Marlin/Marlin_main.cpp#L3166-3189).

> I'm a little shocked that this has gone unnoticed. Thirty-three millimeters should be enough to throw off any calibration. I'd be interested to see what exactly is going on here and why this hasn't bubbled up before.

> In the short term, I'm going to just tweak it in the Marlin firmware and make a pull request, but honestly in the long run I'm probably going to rewrite my own firmware from scratch. I've got some ideas that I'd like to put into practice, and after all, I'm really only building a 3D printer as a playground for my engineering ideas!

> Naturally, I'll be posting code to GitHub as the project progresses. I'm going to be presenting my printer at work in a few weeks, so my goal is to get it at least calibrated before then. My bare-minimum goal is to have it functioning as a plotter by the time I present.

The function in question is correct. Marlin uses `delta_radius` to refer to (the distance from the platform's center to a tower) minus (the distance from the tower to the arm's point of rotation) minus (the radius from the center of the platform to the arm's connection on the platform). This adjusted value does in fact take into account the fact that the three arms do not meet directly at the print head. I misread the source code and, as a result, made a fool of myself.
