---
title: "Baricentric Coordinate"
date: 2018-08-04 09:35:00 0000
categories: blog, snippets
---
# Baricentric Coordinate

To evaluate a random coordinate inside a quad:
{% highlight c++ linenos %}
template <typename T>
T baricentricCoordinateSampling(const T& p0, const T& p1, const T& p2, const T& p3)
{
    auto u = drand48(); 
    auto v = drand48();
    auto a = (1 - u) * v;
    auto b = (1 - u) * (1 - v);
    auto c = u * (1 - v);
    auto d = u * v;

    return (a * p0) + (b * p1) + (c * p2) + (d * p3);
}
{% endhighlight %}

But, because not all faces can be quads, here the triangular version of the same function:

{% highlight c++ linenos %}
template <typename T>
T baricentricCoordinateSampling(const T& p0, const T& p1, const T% p2)
{
    auto u = drand48();
    auto v = drand48();
    if (u + v > 1)
    {
        u = 1 - u;
        v = 1 - v;
    }
    auto a = u;
    auto b = v;
    auto c = (1 - u - v);

    return (a * p0) + (b * p1) + (c * p2);
}
{% endhighlight %}

In both examples \(T\) is a mathematical vector object, defined in \(R^n\) with sum and scalar multiplication.
$$T = \sum_{i=0}^{n}{\lambda_{i}\vec{x_{i}}}$$

