---
title:  "Project Progress and Next Steps"
date:   2021-03-08 15:20:00 +0100
categories: blender
---

## Progress update

Since the last time i talked about our project's progress we have managed to modify Cycles so that it only renders areas of the screenspace which correspond to 2D bounding boxes of geometry in the scene:

<img src="https://pascalhann.github.io/pascals_devblog/assets/Cycles.gif" alt="Modified Cycles showcase">

For now, there is no logic in place, that checks which of the geometry has actually changed and we also have not implemented any method to merge the incremental rendered parts of the image with a full base render of the whole image yet.

## How did we achieve this?

We implemented our method to calculate 2D bounding boxes from 3D bounding boxes ([From this post](https://pascalhann.github.io/pascals_devblog/blender/2021/01/18/Calculating-2D-Bounds.html)) in the method **/render Scene.cpp::device_upodate** directly after the computation for the 3D bounds is finished.  
For this calculation to function properly we need the matrix **"worldToRaster"** from the camera, which is needed to compute the corresponding point of any given point in 3D space, in raster space. Unfortunately, even though this matrix has been calculated at this point and we do have access to it, there is some logic in place that seems to have to do with resolution refinement, that causes the matrix to change value drastically during continuous rendering steps.  
Because of this, we changed the code, so that a second **"worldToRaster_full"** matrix is calculated which always corresponds to the full resolution.

With all this, we have access to the up to date 2D bounding boxes of all geometry in the scene in addition to the to the 2D bounding box of the previous frame.  
We use this information in our modified tile generation. Rather then splitting the whole image into an equal amount of tiles, we generate tiles only where a 2D geometry bounding box lies. We generate multiple overlapping tiles to achieve a finer level of render through multiple passes.

One problem to note is, that the tile generation happens before the bounding boxes are updated in the Cycles pipeline. Because of this we lag behind one step in our renderings, because the bounding box information we use for tile generation is still from the last frame. As of now, we have not found any way to remedy this situation.

## Next Steps

As our next step, we decided to try to implement a logic to merge our partly rendered images, with full base renders of the whole image. The idea is to render one base image at the start or maybe at common intervalls and merge these with our partly rendered images whenever a change in the geometry occurs.

Aside from that, we also need to implement a way to detect changes within geometry, so that we can only render the parts of the scene that have actually changed. This, however, should not pose to much of a problem since we only have to compare our old bounding boxes with the new ones and see if there are any changes.