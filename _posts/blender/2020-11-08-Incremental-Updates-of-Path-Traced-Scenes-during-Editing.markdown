---
title:  "Incremental Updates of Path-Traced Scenes during Editing"
date:   2020-11-08 18:33:24 +0100
categories: blender
---

In order to finish my bachelors degree at TU Vienna, I have to write a work on a bachelor thesis. I applied for the topic "Incremental Updates of Path-Traced Scenes during Editing" at the beginning of september and got it together with another colleague.

The idea is to modify the popular 3D modeling software [blender](https://www.blender.org/) in order to allow incremental rendering updates during live editing, when using the ray-tracing rendering engine [cycles](https://docs.blender.org/manual/en/latest/render/cycles/introduction.html).

In order to achieve this, we have to implement an algorithm that recognizes the parts of the screen that get affected the most by a given action, like for example moving an object, so that we can only rerender these specific parts while the others get rendered in the background. 

We hope that this logic makes the editing experience smoother for the artist and our work shall contain a small user study at the end to reflect on this goal.

Most of the posts on this blog will probably revolve around this topic for the next year or so.