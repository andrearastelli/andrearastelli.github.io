---
title: "Water ripple effect in p5.js"
excerpt: "One of the simplest algorithms to use to show user interaction and pixel operations."
tags: 
  programming
  p5js
  javascript
date: 2020-05-18 18:26:00 0000
categories: programming
---
<iframe width="600" height="320" style="border:none;" src="https://editor.p5js.org/andrearastelli/embed/5J9UCne9r"></iframe>
[Link to the source code here](https://editor.p5js.org/andrearastelli/sketches/5J9UCne9r)

# Water ripples
This is a super simple implementation of the "water ripple" effect, inspired by [Daniel Shiffman's Coding challenge](https://thecodingtrain.com/CodingChallenges/102-2d-water-ripple.html) on the same topic.

Core of the algorithm is:
- two matrices of numbers, one for the current instant and one for the previous.
- A dampening factor (could be a value between 0.0 and 1.0, but the best visual results happens between 0.75 and 0.99999).
- A "propagation" process that picks the value from the previous instant and mixes them with the current instant.

The _propagation process_ can be something like:

~~~ c++
current[x][y] = (
        previous[x-1][y] + 
        previous[x+1][y] +
        previous[x][y-1] +
        previous[x][y+1]
    ) / 2.0 - current[x][y];

current[x][y] *= dampening;
~~~

This will pick the values in the cell above, below, on the left and on the right of the current cell.

    +-------+-------+-------+
    |       |       |       |
    |       | x,y-1 |       |
    |       |       |       |
    +-------+-------+-------+
    |       |       |       |
    | x-1,y |  x,y  | x+1,y |
    |       |       |       |
    +-------+-------+-------+
    |       |       |       |
    |       | x,y+1 |       |
    |       |       |       |
    +-------+-------+-------+

# Variations
While experimenting with this algorithm, I've tried doing some changes in the way the values are picked, by picking the diagonal values as well, but decreasing their value by a factor of 2:

~~~ c++
current[x][y] = (
        previous[x-1][y] + 
        previous[x+1][y] +
        previous[x][y-1] +
        previous[x][y+1] +
        (
            previous[x-1][y-1] +
            previous[x-1][y+1] +
            previous[x+1][y-1] +
            previous[x+1][y+1]
        ) * 0.5
    ) / 3.0 - current[x][y];

current[x][y] *= dampening;
~~~

And here the result:
<iframe width="600" height="320" style="border:none;" src="https://editor.p5js.org/andrearastelli/embed/AjXguvKkR"></iframe>
[Link to the source code here](https://editor.p5js.org/andrearastelli/sketches/AjXguvKkR).

This result in a lightly round-ish shape of the wave generated, at the expense of 4 more values to be evaluated.

With a modern machine this algorithm still runs at about 60fps[^p5js-framerate-limit].

[^p5js-framerate-limit]:
    P5.js has a framerate limit of 60 fps by default.
    This can be confirmed by running [this script](https://editor.p5js.org/andrearastelli/sketches/9uR3ZYInN) where every 10 frames the current framerate is updated, and every 1000 frames the max framerate is reset (this allow a window of 1000 frames circa to see what the max framerate actually is).
    The `random()` function is called once per frame, so the computation impact is very very low, and the text draw functions can still be fast enough to be executed, in a sketch of 200x200 pixels, way faster than 60fps in any other graphical library (processing, for example).



