---
title: "Fireworks over planet"
excerpt: "A simple P5.js sketch showing how to shoot fireworks using forces from a sphere"
classes: wide
tags: 
  javascript
  p5.js
  snippets
date: 2020-05-13 8:55:00 0000
categories: programming
---

<iframe width="600" height="600" frameborder="0" src="https://editor.p5js.org/andrearastelli/embed/KlWtDifEZ"></iframe>

[Full window version](https://editor.p5js.org/andrearastelli/present/KlWtDifEZ)

This was quite an interesting "challenge", inspired by [The Coding Train](https://www.youtube.com/channel/UCvjgXvBlbQiydffZU7m1_aw).

Here the original video, where Daniel Shiffman goes thru simple physics behaviour in order to show 2D first, then 3D, fireworks being shoot from the bottom of the frame and exploding somewhere around the upper section of the frame.

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/CKeyIbT3vXI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In my version, I wanted to explore more the physics behaviour by generating the fireworks from a sphere (instead of an ideal ground plane like $$y=0$$).

This presents several challenges:
- The fireworks are being shoot from the surface of the planet
- A radial force must be applied, such that every new firework is being "attracted" by the planet (it's center)
- Same force applies to the exploding particles
- All the drawing must happen, while rotating around the planet (the idea was to give the impression of a view around the world maybe at new year).

There are still some things to fix and improve, like a better force system and maybe some better drawing process to better show the rotation around the planet.

Was also thinking to move the whole drawing process from P5.js to Three.js in the hope to gain some performance boost using some Three.js tricks (workers?)

Would be interesting to try the 2D Fireworks version in some shader language, to experiment with a different approach to the problem.
