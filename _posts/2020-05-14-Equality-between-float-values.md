---
title: "Equality between two float values"
excerpt: "Notes about checking if two float values are the same"
classes: wide
tags: 
  algorythms
  programming
date: 2020-05-14 8:21:00 0000
categories: programming
---
When checking if two values are the same, mathematically speacking we go with a simple equality condition: `if a = b: ...`.
This can be a valid solution in general math, where $$2.01$$ needs to be equal to some expression like $$1 + 1 + 0.01$$.

When writing code, things become a little more complex.

Don't want to go deep into float arithmetic in programming - this is just a personal note, but to keep this simple, sometimes when writing floating point numbers in code, you don't really get the number you thought to get. For example, doing something like `float x = 2.01;` can be evaluated as `x = 2.01000001` or `x = 2.00999999`.
This may not look like a big deal, but when processing those numbers over and over again, we can accumulate this error and have a resulting calculation that is completely off compared to the pure mathematical result.

To approach this problem when checking if two floating numbers are the same, instead of using the `==` operator, can be wise to calculate the error resulting from the absolute difference between the two numbers.

~~~ c++
float error = abs(a - b);
if (error <= 0.000001)
{
    # magic...
}
~~~

This way we use a more rigorous mathematical approach into veryfying if two real numbers are "_so close to each other that their difference is smaller than any small number_" (in this case _any small number_ is represented by an error threshold value like $$0.000001$$).

