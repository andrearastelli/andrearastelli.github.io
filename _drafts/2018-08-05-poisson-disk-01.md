---
title: "Poisson Disk - 01"
date: 2018-08-05 07:48:00 +0100
categories: blog
tags: snippets
---
Poisson Disk distribution, defines a very pleasing point sampling algorithm, with blue-noise properties.

Sampling points using this system translates into a random distribution where each point is defined with a radius. The circle defined by the point center and the radius, is a region where no other point can be placed into.

Multiples algorithm exists with the ability to generate such distribution, but only some of them have proper application for both 2D and 3D applications, like sampling Poisson Disks over a 3D mesh.

This kind of distribution is one of the most complex to handle, because there exists multiple problems that needs to be handheld properly.

The first problem consists of evaluating the poygonal mesh so that each triangle can be properly sampled using any distribution.

Because we need to start from a single triangle, sampling a single point, the best solution could be to randomly pick a face, and randomly pick a sample on that face.

{% highlight %}
randomFaceId = randInt( 0, numFaceIds - 1 );
P0, P1, P2 = getTrianglePointsFromFaceId( randomFaceId );
P = randomSampleUsingBaricentricCoordinate( P0, P1, P2 );
{% endhighlight %}

This process, once executed, cannot be used anymore, because from this moment on, all the other samples will be extracted starting from this first point.

This means that we need a way to get all the connected faces from a specified one (most 3D polygon API should give us already that).




