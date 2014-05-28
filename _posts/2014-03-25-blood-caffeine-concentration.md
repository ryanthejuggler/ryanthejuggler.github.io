---
layout: post
title: "Blood caffeine concentration"
date: 2014-03-25 10:53:00
categories: math
---
Looking through an old server from a hosting provider I'm migrating away from, I found an old algorithm I used to estimate blood caffeine concentration.

~~Just a word of warning, I've lost the link to the paper I drew this information from. If someone wants to find where my assumptions came from I'll buy you a coffee.~~ **Update**: I've found the reference in an old notebook! [Consumption and Metabolism of Caffeine -- Biology Online](http://www.biology-online.org/articles/actions_caffeine_brain_special/consumption_metabolism_caffeine.html).

Times here are specified to be in seconds from any arbitrary epoch. Since only time differences really factor into the equations it doesn't matter what your epoch is.

Let $L$ be a constant value allowing us to approximate a person's blood volume based on their body mass. I found $L = 0.007 \mathrm{\frac{L}{kg}}$.

Let $M$ be your mass and $t$ be the current time. Let $\lambda$ represent the half-life of caffeine in the body and let $\alpha$ represent an absorption coefficient. My estimates for these values are $\lambda = 5\,\mathrm{hours}$ and $\alpha = 1\,\mathrm{hour}$.

Let $a\_i$ and $t\_i$ be series representing doses of caffeine. $a\_i$ represents the amount of the dose while $t\_i$ represents the time of the dose.

The amount of caffeine in your bloodstream can be approximated by:

$$
\sum\_i \left(
\frac{ a\_n \exp\left({\frac{-(t-t\_n)\ln 2}{\lambda}}\right) \left( 1 - \exp\left(\frac{(t-t\_n)\ln 0.05}{\alpha}\right) \right)}
{ML}
\right)
$$

~~Again, **I've lost the links to my original references.**~~ Even though I've found the references, that's not to say that the assumptions I made are any good. I'm not a doctor. Use at your own risk.

Below is the original algorithm. I hope I've faithfully translated it back to abstract mathematics. Later I'll integrate it so that we can evaluate the effects of a slowly-consumed cup of coffee rather than considering consumption as a point event.

{% gist 9763822 %}
