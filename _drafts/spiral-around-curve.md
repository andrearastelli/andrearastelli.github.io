---
title: "Spiral around curves in Houdini"
tags: snippets
categories: houdini
---
During my last trip to Rome, a friend of mine asked me some help for creating a spiraling curve around another (simpler, probably more linear) curve he wanted to use as a guide.

And he wanted to do everything in Houdini, VEX, of course.

Now, I barely know Houdini, so I can use this as an excuse to start playing with it.

First I created a 2 point curve (a start point and and end point).
This way, for me, was simple to understand and debug the problem.

Then I created a "point wrangler" node, to loop through the 2 points, and started to create simple circles around them given a radius.

Simple enough, I think.

{% highlight c %}

{% endhighlight %}

