---
title: "Baricentric Coordinate"
date: 2018-08-04 09:35:00 0000
tags: snippets
classes: wide
categories: programming
---
A couple of useful ways to extract random points in a n-dimensional triangle or quad, using only 2 random values between 0 and 1.

To evaluate a random coordinate inside a quad:

~~~ c++
template <typename T>
T baricentricCoordSampling(const T& p0, const T& p1, const T& p2, const T& p3)
{
    auto u = drand48(); 
    auto v = drand48();
    auto a = (1 - u) * v;
    auto b = (1 - u) * (1 - v);
    auto c = u * (1 - v);
    auto d = u * v;

    return (a * p0) + (b * p1) + (c * p2) + (d * p3);
}
~~~

But, because not all faces can be quads, here the triangular version of the same function:

~~~ c++
template <typename T>
T baricentricCoordSampling(const T& p0, const T& p1, const T& p2)
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
~~~

In both examples $$T$$ is a mathematical vector object, defined in $$R^n$$ with sum and scalar multiplication.

