---
title: "Attractor e Repulsor"
excerpt: "Playing with attractor and repulsor forces in a particle field."
tags: 
  programming
  p5js
  javascript
  physics
date: 2020-05-27 07:41:00 0000
categories: programming
---
Super small experiment to try particle behaviour with attractors and repulsors.

Another experiment that uses Daniel Shiffman's [Attraction Repulsion Coding Challenge](https://thecodingtrain.com/CodingChallenges/056-attraction-repulsion.html) as a source of inspiration.

Basic idea is to build a toy particle simulator with very simple physics and where is possible to add components to change the particles behaviour (ie: attractors and repulsors).

# Particles
When building a particle-_something_, the first thing to create is the particle object.

~~~ javascript
class Particle
{
    constructor()
    {
        this.pos = createVector(random(100, 300), random(100, 300));
        this.ppos = this.pos.copy(); // only for drawing the particle line

        this.vel = p5.Vector.random2D();
    }

    update()
    {
        this.ppos = this.pos.copy();
        this.pos.add(this.vel);
    }

    applyForce(force)
    {
        this.vel.add(force);
    }

    show()
    {
        noFill();
        stroke(255);
        strokeWeight(2);
        line(this.ppos.x, this.ppos.y, this.pos.x, this.pos.y);
    }
}
~~~

This `Particle` object is incredibly simple, and there is pretty much no logic in it.

When created, will spawn a particle in a random position around the center of the canvas, and because each particle is created with a random initial velocity, the particle will move in a random direction at a constant speed.

![single particle with random velocity](/assets/images/2020-05-27-attractor-repulsor/particle_01.gif){: .align-center}

Pretty boring.

And will remain boring even if we add more than one particle, as there is no interaction whatsoever that will make this any more interesting.

![multiple particles with random velocity](/assets/images/2020-05-27-attractor-repulsor/particle_02.gif){: .align-center}

> I mean, _boring_ as in: "there is nothing that happens, except for a bunch of points that moves outward from the center of the canvas".[^01]

[^01]:
    The gifs shows a loop for the same animation, the actual script, if not interrupted, will see the particles move at constant speed always in the same direction, without ever stopping, till the end of time.

# Interaction and attractors
In order to make things interesting (and super complex in due time) we can start to define specific behaviours.

First let's identify how complex we want the interaction to be:
1. We can have random elements, `Attractor` objects, that are _things_ the particle will gravitate towards.
2. We can make `Repulsor` objects that behaves in the opposite ways than an `Attractor`, and will steer the particles _away_ from them.
3. We can say that _all the particles are attractors for all the particles_.
4. We can define **flow fields** that defines how a particle interact with the environment in order to reach a specific destination.
5. We can assign a _life_ to a particle and start to update the movement based on some function of it (for example, the older a particle gets, the slower it goes; each particle can have a fixed life and when that age if reached that particle dies; and so on...).

## Working with the `Attractor` object

![single attractor with 10 particles that interact with it](/assets/images/2020-05-27-attractor-repulsor/particle_03.gif){: .align-center}

How to define an attractor is pretty straightforward.
In my example and `Attractor` object doesn't exists yet, and the particles all gravitate towards vectors in space.

~~~ javascript
// In the setup function
function setup()
{
    // code to setup the canvas and the particles
    // ...
    // ...
    // Setup for the attractors
    for (let i=0; i<NUM_ATTRACTORS; i++)
    {
        const x = width/2;
        const y = height/2;
        const offset = 50;
        const xpos = random(x-offset, x+offset);
        const ypos = random(y-offset, y+offset);
        const v = createVector(xpos, ypos);
        
        attractors.push(v);
    }
}
~~~

And, in the `Particle` class I've added a function that scans over a list in order to update the direction of each particle based on the sum of all the points in the input list:

~~~ javascript
class Particle
{
    // Code that you already know

    applyField(targets)
    {
        let fSum = createVector();
        for (let target of targets)
        {
            const t = target.copy();
            t.sub(this.pos);
            t.normalize();
            fSum.add(t);
        }
        fSum.normalize();
        this.applyForce(fSum);
    }
}
~~~

$$\vec{f_{tot}} = \sum_{i}{\vec{f_i}}$$

As the formula shows, the forces are all added together without any major thought about how each object should interact with each other.

This means, among other things, that each attractor has exactly the same influece as any other attractor, generating behaviours always similar and always pretty much non-interesting[^2].

[^2]:
    By _non interesting_ I mean that every single repetition of the simulation will generate almost the same behaviour for the particles, regardless of where the attractors are.

![multiple generations of attractors and particles with attractor influencing the particles in the same way regardless of the distance or other properties](/assets/images/2020-05-27-attractor-repulsor/particle_04.gif){: .align-center}

# Adding more complexity to the `Attractor` object
In order to have a more complex behaviour we could say that each attractor has a mass, and then use the Newton's Gravitational Laws to evaulate how this mass is going to affect each particle movement:

$$F = G \frac{m_1 m_2}{d^2}$$

This means that we can construct an `Attractor` object as follows:

~~~ javascript
class Attractor
{
    constructor()
    {
        const x = random(width/2-offset, width/2+offset);
        const y = random(height/2-offset, height/2+offset);
        this.position = createVector(x, y);

        this.mass = random(1, 10);
    }
}
~~~

The position of the attractor is fixed in the canvas, and the mass is any random value in the range $$[1, 10)$$.

With this change, we need to re-evaluate how the force is being calculated for each particle that interact with the attractor, and a way to do this is by calculating the distance between a particle and an attractor, consider a particle with mass $$m = 1$$, and have a resulting formula similar to:

$$F = G\frac{m_a * 1}{d^2}$$

Problem is: the force $$F$$ calculated this way is a scalar value and not a vector.
We need to apply this force to the normalized vector that points from the particle $$\vec{p_p}$$ to the attractor $$\vec{p_a}$$:

$$\vec{v} = \left\lVert\vec{p_p} - \vec{p_a}\right\rVert$$

~~~ javascript
class Particle
{
    applyForceField(attractors)
    {
        const forceSum = createVector();
        for (let attractor of attractors)
        {
            const distance = this.position.dist(attractor.position);
            // Or use the p5.Vector.dist(v1, v2)

            const force = G * (attractor.mass * 1) / (distance * distance);

            const dir = p5.Vector.sub(this.position, attractor.position);
            dir.normalize();
            dir.setMag(force);

            forceSum.add(dir);
        }

        this.applyForce(forceSum);
    }
}
~~~

Note how the $$G$$ value is not defined in the `applyForceField` function.
This because my idea was to have a global constant `G` that basically act as a multiplier.
This because in the pixel space the simulation lives into has quite a restricted area for the simulation to be drawn into.

If we're not careful with the `G` value (let's say we leave `G = 1`), we can have _gigantic_ forces compared to the space those forces are being applied into.
