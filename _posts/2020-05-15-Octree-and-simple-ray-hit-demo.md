---
title: "Simple Raytracing with octrees"
excerpt: "How to build a simple octree implementation to quickly raytrace a (complex) scene."
toc: true
toc_sticky: true
tags: 
  demo
  programming
  p5js
  processing
  javascript
date: 2020-05-14 8:21:00 0000
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

And then the basic math and algorithm used to process the scene and trace rays from an ipotetical camera and thru an ipothetical image grid
