---
title: "Simple Raytracing with octrees - Part 1"
excerpt: "Part 1 of a series about writing a simple raytracer in p5.js and trying to demonstrate the use of octree to speed up the generation of the final image."
toc: true
toc_sticky: true
tags: 
  demo
  programming
  p5js
  processing
  javascript
  raytracing
date: 2020-05-16 19:40:00 0000
categories: programming
---
# Introduction
In this demo there are several points I would like to touch in order to proceed with the implementation of this simple raytracing algorithm.

The first is related to how to proceed with the nahive implementation and store simple objects (spheres) into a flat array that must be iterated over and over, for each pixel, in order to know if the ray thru that pixel is hitting a sphere, multiple spheres or anything.
If multiple spheres are being hit, only the closest one to the camera must be evaluated.

This basically means raytracing with one bounce and one sample per pixel (spp) any given scene.

Then I will proceed with the octree implementation and store the scene content into this data structure.
Next step will be to implement a ray-hit implementation that uses the octree and the ray to figure out when any given ray hits the octree and what nodes in the tree are actually being touched.

Of course, for simple scenes (very few spheres in this demo) there will be almost no difference between the flat array and the octree (when the octree don't perform worse than the flat array).
But things change as soon as we start to add more and more elements in the scene. At some point the octree will be order of magnitudes faster than a flat list of elements.

This will help a lot with the examples in this demo, written in javascript (specifically p5.js) will not have any smart optimizations (other than the octree) and the overall implementation will be as nahive as possible to achieve the desired result.

# How to write a simple raytracer
Let's start from the beginning, and figure out if p5.js provides us the basic elements needed to build something simple.

We will need:
- A vector object (p5.js has a `p5.Vector` class)
- A matrix object (p5.js has a `p5.Matrix` class)
- An easy way to draw and navigate in the 3D scene (p5.js has several 3D classes and functions useful for this task)
- The ability to write into an image-like pixel array (p5.js has several objects to do this, a graphics class and an image class)

And then the basic math and algorithm used to process the scene and trace rays from an ipotetical camera and thru an ipothetical image grid into the scene and perform a hit with any intersecting object, and return the closest object info.

As a resource for the raytracing math and algorithm, I'll be using the amazing [Raytracing in one weekend](https://raytracing.github.io/) book serie from [Peter Shirley](https://twitter.com/Peter_shirley).

Specifically the chapters about:
- [Sending rays into the scene](https://raytracing.github.io/books/RayTracingInOneWeekend.html#rays,asimplecamera,andbackground/sendingraysintothescene)
- [Ray-Sphere Intersection](https://raytracing.github.io/books/RayTracingInOneWeekend.html#addingasphere/ray-sphereintersection)
- [Ray intersection with an AABB](https://raytracing.github.io/books/RayTracingTheNextWeek.html#boundingvolumehierarchies/rayintersectionwithanaabb)

The last of which will be useful in the octree implementation step.

# Let's start
## Scene setup
Let's start by setting up a simple scene in the [p5.js online editor](https://editor.p5js.org/).

{% highlight javascript %}
let scene = [];

function setup() {
  createCanvas(400, 400, WEBGL);
  
  for (let i=0; i<20; ++i)
  {
    const sphereObj = {
      "x": random(-width/2, width/2),
      "y": random(-width/2, width/2),
      "z": random(-width/2, width/2),
      "radius": random(10, 50),
      "color": color(random(255), random(255), random(255))
    };
    scene.push(sphereObj);
  }
  
}

function draw() {
  background(220);
  
  orbitControl();
  
  noStroke();
  push();
  for (let i=0; i<scene.length; ++i)
  {
    const sphereObj = scene[i];
    push();
    translate(sphereObj.x, sphereObj.y, sphereObj.z);
    fill(sphereObj.color);
    sphere(sphereObj.radius);
    pop();
  }
  pop();
}
{% endhighlight %}

[Link to the code](https://editor.p5js.org/andrearastelli/sketches/sIhVPvz7y)

<iframe style="outline: none; border: none;" width="400" height="400" src="https://editor.p5js.org/andrearastelli/embed/sIhVPvz7y"></iframe>
(click and drag the mouse to navigate in the scene)

This very very basic scene setup don't use - yet - any object like `Sphere` or other primitive types to represent each sphere being drawn.

Right now every sphere information: color, position in 3D space and radius, is stored in a very simple dictionary object:

{% highlight javascript %}
const sphereObj = {
  "x": random(-width/2, width/2),
  "y": random(-width/2, width/2),
  "z": random(-width/2, width/2),
  "radius": random(10, 50),
  "color": color(random(255), random(255), random(255))
};
{% endhighlight %}

This can be improved by moving the sphere representation into it's own object.

Then we have the scene, that currently is represented by a simple array.
We'll see how to better extend the scene representation to be an object that will contains both the objects and the algorithm to process ray evaluations.

## Sphere Primitive

{% highlight javascript %}
class Sphere
{
    constructor(position)
    {
        this.radius = random(10, 50);
        this.color = color(random(255), random(255), random(255));
        if (position)
            this.position = position;
        else
            this.position = createVector();
    }

    show()
    {
        push();
        translate(this.position.x, this.position.y, this.position.z);
        fill(this.color);
        sphere(this.radius);
        pop();
    }
}
{% endhighlight %}

This very simple sphere representation can be later extended to include ray-sphere intersection and other functions that can be useful for evaluating the sphere in the scene.

With this change the scene generation now is something similar to:
{% highlight javascript %}
let scene = [];

function setup() {
  createCanvas(400, 400, WEBGL);
  
  for (let i=0; i<20; ++i)
  {
    const position = createVector(
      random(-width/2, width/2),
      random(-width/2, width/2),
      random(-width/2, width/2)
    );
    
    scene.push(new Sphere(position));
  }
  
}

function draw() {
  background(220);
  
  orbitControl();
  
  noStroke();
  push();
  for (let obj of scene)
  {
    obj.show()
  }
  pop();
}
{% endhighlight %}
[Link to the code](https://editor.p5js.org/andrearastelli/sketches/r5pTjWAT7)

The code is a little less, as we moved the logic for drawing the spheres and generating the actual sphere object into the `Sphere` class.


## Considerations before moving forward
Up until now, the whole setup is only focused on p5.js setup to draw a super simple set of spheres in a 3D environment (with no shading, only a flat material).

From this moment on, instead, the focus will be on generating the raytraced image, and then on the octree implementation and possible speedup into rendering the image.

This implies that, for debug purposes, I needed to separate the 3D and the 2D rendered preview in such a way that would be easy to see what is happening in the 3D scene and, at the same time, what the rendered image looks like.

In order to do that, I'll be using a [`p5.Graphics`](https://p5js.org/reference/#/p5.Graphics) object and a [`p5.Image`](https://p5js.org/reference/#/p5.Graphics) object. This kind of objects allows the creation of a graphics context in addition to the main one provided by the basic `createCanvas()` function.

I'll be using those objects, one for the 3D scene preview and one for the actual raytraced image.

The idea is to draw those two graphics context, one next to the other, so will be clear what the raytracer is doing in the 3D scene and how the final image genration looks like in the preview.

[Here the updated code used to view both the 3D scene and the future rendered image](https://editor.p5js.org/andrearastelli/sketches/vA3zMVQ9h)

<iframe width="600" height="300" style="border:none; outline:none" src="https://editor.p5js.org/andrearastelli/embed/vA3zMVQ9h"></iframe>

(The scene navigation will happen in a limited way now - maybe I'll spend more time on this later)


# Image generation
Now that we have a super basic scene, with a simple Sphere primitive, we can start generating the actual image.

> before doing so, I've rearranged some of the code to keep things clean, as the multiple images started to make the whole thing a little messy.
> [Link to the updated scene is the same as the image generation code](https://editor.p5js.org/andrearastelli/sketches/35yunHB2g)

The [Raytracing in one weekend book has a neat piece of code](https://raytracing.github.io/books/RayTracingInOneWeekend.html#thevec3class/colorutilityfunctions) that is used to generate an image.

Implementing that code in our demo will generate this result:

{% highlight javascript %}
function drawImage(g2d)
{
  g2d.loadPixels();
  for (let j=0; j<cHeight; ++j)
  {
    for (let i=0; i<iWidth; ++i)
    {
      let c = color(
        (float(i) / (iWidth - 1)) * 255,
        (float(j) / (cHeight - 1)) * 255,
        0.25 * 255
      );
      g2d.set(i, j, c);
    }
  }
  g2d.updatePixels();
}
{% endhighlight %}

<iframe width="400" height="200" style="outline:none; border:none;" src="https://editor.p5js.org/andrearastelli/embed/35yunHB2g"></iframe>
[Link to the code](https://editor.p5js.org/andrearastelli/sketches/35yunHB2g)

Main differences are:
- Because we have an `p5.Image` object, we need to load the pixels to be then able to write into them using the `g2d.set(x, y, color)` function.
- Instead of using a `write_color` function, we're translating the color into the corresponding RGB [0, 255] values immediately.

Has to be kept in mind that the image generation process will become **a lot** slower when we will add more rendering features into it.[^speed-up-image-generation]


### What is this thing doing?
Well.. we are basically considering the image as a set of "slots" (the pixels) distributed in rows and columns.
The number of rows and columns corresponds to the width and height of the image in pixels, and by iterating over every single one of them, we can evaluate some procedural value to assign to every pixel in order to start "drawing" in the image.

This is the very first step over iterating over the scene starting from the image "plane" (more on this when we will start talking about cameras).


## The Ray object and sending rays into the scene
So, not the fun starts.
We are about to start working the the proper raytracing and what better place to start if not the Ray class?

{% highlight javascript %}
class Ray
{
  constructor(origin, direction)
  {
    this.origin = origin;
    this.direction = direction;
  }
  
  at(t)
  {
    return this.origin.add(this.direction.mult(t));
  }
}
{% endhighlight %}

> Due to javascript quirks, there is a massive difference between the C++ version of this code. I've decided to take the "simplest" approach possible, where I'll be discarding the `origin()` and `direction()` methods, and using the attributes, when needed, directly.
> Also the `at()` method uses the `p5.Vector` methods for the `this.origin` and `this.direction` in order to generate the point position along the ray.
 
Now that we have the `Ray` class in place, we can start raytracing, by updating the `drawImage` function as follows:

{% highlight javascript %}
function ray_color(ray)
{
  const direction = ray.direction;
  direction.normalize();
  const t = 0.5 * (direction.y + 1.0);
  const c0 = createVector(1.0, 1.0, 1.0);
  c0.mult(1.0 - t);
  const c1 = createVector(0.5, 0.7, 1.0);
  c1.mult(t);
  const final_vec_color = p5.Vector.add(c0, c1);
  final_vec_color.mult(255);
  
  return color(final_vec_color.x, final_vec_color.y, final_vec_color.z);
}


function drawImage(g2d)
{
  const origin     = createVector();
  const horizontal = createVector(4.0, 0.0, 0.0);
  const vertical   = createVector(0.0, 2.25, 0.0);

  const lower_left_corner = p5.Vector.sub(origin, p5.Vector.mult(horizontal, 0.5));
  lower_left_corner.sub(p5.Vector.mult(vertical, 0.5));
  lower_left_corner.sub(createVector(0.0, 0.0, 1.0));
  
  g2d.loadPixels();
  for (let j=cHeight-1; j>=0; --j)
  {
    for (let i=0; i<iWidth; ++i)
    {
      const u = float(i) / (iWidth - 1);
      const v = float(cHeight - j) / (cHeight - 1);
      
      const ray_direction = lower_left_corner.copy();
      
      ray_direction.add(p5.Vector.mult(horizontal, u));
      ray_direction.add(p5.Vector.mult(vertical, v));
      
      const r = new Ray(origin, ray_direction);
      const c = ray_color(r);

      g2d.set(i, j, c);
    }
  }
  g2d.updatePixels();
  noLoop();
}
{% endhighlight %}


# First raytraced image of our scene
Aaaaand... here it is:

<iframe width="400" height="200" style="outline:none; border:none;" src="https://editor.p5js.org/andrearastelli/embed/qW6Ny6prE"></iframe>

{% highlight javascript %}
function ray_color(ray)
{
  for (let sphere of scene)
  {
    if (hit_sphere(sphere, ray))
    {
      return sphere.color;
    }
  }
  
  const direction = ray.direction;
  direction.normalize();
  
  const t = 0.5 * (direction.y + 1.0);
  const c0 = createVector(1.0, 1.0, 1.0);
  c0.mult(1.0 - t);
  const c1 = createVector(0.5, 0.7, 1.0);
  c1.mult(t);
  const final_vec_color = p5.Vector.add(c0, c1);
  final_vec_color.mult(255);
  
  return color(final_vec_color.x, final_vec_color.y, final_vec_color.z);
}


function drawImage(g2d)
{  
  g2d.loadPixels();
  for (let j=cHeight-1; j>=0; --j)
  {
    for (let i=0; i<iWidth; ++i)
    {
      const u = float(i) / (iWidth - 1);
      const v = float(cHeight - j) / (cHeight - 1);
      
      const ray_direction = lower_left_corner.copy();
      
      ray_direction.add(p5.Vector.mult(horizontal, u));
      ray_direction.add(p5.Vector.mult(vertical, v));
      
      const r = new Ray(origin, ray_direction);
      const c = ray_color(r);

      g2d.set(i, j, c);
    }
  }
  g2d.updatePixels();
}

function hit_sphere(sphere, ray)
{
  let oc = p5.Vector.sub(ray.origin, sphere.position);
  let a = p5.Vector.dot(ray.direction, ray.direction);
  let b = 2.0 * p5.Vector.dot(oc, ray.direction);
  let c = p5.Vector.dot(oc, oc) - sphere.radius * sphere.radius;
  let discriminant = b * b - 4 * a * c;
  
  return discriminant > 0;
}
{% endhighlight %}

The new function `hit_sphere` is the core of the system now as is the one that is actually checking if a specific sphere is being hit at any given point.

Then the `ray_color` function iterates over the scene array and as soon as _a_ sphere is being hit, it's color is returned.

> A very important thing to notice here is that the system doesn't have (yet) any consideration for distance of the spheres to the camera, so the image generated will show the sphere distance as a function of the distance of each sphere object to the beginning of the array.
 
## Raytracing line by line
In order to have an even better understanding of what is going on, this preview of the "live" update of the raytraced scene may be of help:

<iframe width="400" height="200" style="border:none;" src="https://editor.p5js.org/andrearastelli/embed/Oa_c4m69e"></iframe>

I've setup the script such that, for every complete render, a new scene is being setup and the render will start again.

[Here the code](https://editor.p5js.org/andrearastelli/sketches/Oa_c4m69e).


# Next steps 
..part 2 link here..

[^speed-up-image-generation]:
    When writing a raytrace using a programming languages that compiles to machine code, is usually a good idea to split the rendered area into portions that can then be _rendered_ each in a different thread, so the whole process is parallelized.
