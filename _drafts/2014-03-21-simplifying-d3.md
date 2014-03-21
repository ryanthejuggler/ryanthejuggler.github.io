---
layout: post
title: Simplifying D3
date: 2014-03-21 08:21:00
categories: javascript
---

I've been using D3 at work a lot lately and have come to realize that it wants to be its own language. Its seemingly infinitely-chained method calls are awkward at best and disastrous at worst.

> Tangent: I like to chain my methods by putting each dot at the start of a new line, like so:
> {% highlight javascript %}
getAnObject()
  .thenDoThis()
  .andThenDoThis();
{% endhighlight %}
> I was once told not to do that, because my code would be later edited in Eclipse, and someone
> who would be editing it had an auto-format configuration that would somehow break this code. This did not make me very happy.

Take this, for instance, excerpted from [one of Mike Bostock's examples](http://bl.ocks.org/mbostock/9078690):

{% highlight js %}
svg.selectAll(".node")
    .data(nodes(quadtree))
  .enter().append("rect")
    .attr("class", "node")
    .attr("x", function(d) { return d.x0; })
    .attr("y", function(d) { return d.y0; })
    .attr("width", function(d) { return d.y1 - d.y0; })
    .attr("height", function(d) { return d.x1 - d.x0; });
{% endhighlight %}
