---
title: "Baricentric Coordinate"
date: 2018-08-04 09:35:00 0000
categories: blog, snippets
---
# Baricentric Coordinate
{% highlight c++ linenos %}
template <typename T>
T baricentricCoordinateSampling(const T& p0, const T& p1, const T& p2, const T& p3)
{
    auto u = drand48();
    auto v = drand48();

    return (1 - u) * v * p0 + (1 - u) * (1 - v) * p1 + u * (1 - v) * p2 + u * v * p3;
}
{% endhighlight %}
