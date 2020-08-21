---
title: "State of the pipeline in VFX"
excerpt: "How many VFX studios pipelines are *not* for the artists to use."
tags: 
  vfx
date: 2020-08-16 18:26:00 0000
categories: personal
---
Many VFX Studios, around the world, exists for more than a decade now, and many of them have a well established pipeline for the artists to use, every day, in order to generate the awesome images we see in all the movies out in the cinemas every season.

What is a "pipeline" you ask?

Well, in a VFX studio is common to call "pipeline" all the customization and tools on top of the standard tools commercially available (like Autodesk Maya, Foundry Katana or Nuke, and so on) in order to better integrate them into the movie-making workflow that the company decided to have.

On top of this, each department, each artist and each movie, can have it's own very specific way of working in order to deliver the best posible results, and the "pipeline" can come in and help by providing tools, automations and workflows that can be used to "help" the artists.


_Or, at least, this should be the idea._


Yes, because if the overall idea is to build a tool that can help speed up the work or automate it, most of the time everything falls down to the final implementation of the tools.

Don't get me wrong, I know lots of software developers that works on such tools, and *all* of them are super amazing in their daily job.

But, sometimes, the final result is.. well.. lacking something.

For example, let's assume we're in the lighting software team and we need to build "something" that allows the artists to generate better images, faster, and with more control over their work so that they can either hit "render" and generate the final image with no other work on their side, or change all the way down to each single asset the shaders or the textures.

Having such control in a single tool is *not at all an easy thing to achieve*, and most of the time it requires lots of efforth and lots of design - not to mention all the compromises that needs to be had.

One way to implement such tool could be to simply ignore the UX, and give the artists the most complex tool ever designed, where they only have "fine control" over the scene, where they need to learn how to script things in order to change something in the system.

Another way could be to provide no access to the internals of the scene, and 