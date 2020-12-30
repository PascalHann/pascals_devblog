---
title:  "Understanding Cycles part 1"
date:   2020-12-30 14:45:24 +0100
categories: blender
---

It has been a while since the last post so I am writing this one to recollect my findings since then.

## Draw loop update
I have gained further insight in how the code flow looks like when drawing the viewport with Cycles:

* draw_manager.c : DRW_draw_view -> 
* draw_manager.c : DRW_draw_render_loop_ex (render Loop für externe render engines?) ->
* draw_manager.c : drw_engines_init -> 
* external_engine.c : external_engine_init -> 
* draw_manager.c : drw_engines_draw_scene ->
* external_engine.c : external_draw_scene -> 
* ... ?RNA? ... ->

This was how far i was in the last post on this matter. Now comes the Cycles part. I note the subfolder in the Cycles source code in addition to the file and method:

* /blender CCL_api.h (Python API between Blender and Cycles?)
* /blender blender_python : draw_func ->
* /blender blender_session.cpp : draw (draw call in Cycles with width and height) ->
* /render session.cpp : draw ->
* /render session.cpp : draw_gpu ->
* /render buffers.cpp - DisplayBuffer>::draw ->
* /device device.cpp : draw_pixels ->
* glDrawArrays

## Understanding Cycles

Since we decided to implement our project completely within Cycles, my attention has shifted from understanding the Blender source to Cycles source code.

I am especially interested in how the dependency graph is passed from Blender, stored, accessed and updated, since we need that information to determine the most affected regions by recent changes of the viewport.

Here are my findings in no particular order:

There is a graph.cpp in subfolder /render and a node.cpp in /graph but I wasn't able to wrap my head around how they work yet.  
It appears to me, that there are several subclasses of node, like geometry.cpp (/render). These are stored in the dependancy graph and compose the actual data within the graph.

In geometry.h we have a **3D BoundingBox**, we have boolean flags for transform_applied and a **transformation matrix 4x3**. In geometry.h there is also a class called GeometryManager but I am not sure what exactly it does yet.

There are two different 'Graph Compilers' one for osl (open shading language), found in osl.h and one for svm (don't know whta that stands for yet) in svm.h.

In the rna code there is the file RNA_blender_cpp.h which contains a class "Depsgraph", which is a child of Pointer.
It is used several times in blender_session.cpp. The dependancy graph contains, beside the scene, **also a collection of DepsgraphUpdate objects, which contain an id of an object within the graph and how it has been updated: transformation, geometry or shading**.

Here is my current theory on how the dependancy graph gets passed from Blender to Cycles:

* blender calls sync_function in /blender blender_python.cpp with the dependancy graph as a parameter ->
* /blender blender_session.cpp : synchronize ->
* /blender blender_sync.cpp : sync_recalc -> 
* /blender blender_session.cpp : synchronize ->
* /blender blender_sync.cpp : sync_data ->
* /blender blender_object : sync_objects ->
* /blender blender_object : sync_object - a lot of stuff happens here. It seems to me that only objects that are geometry get synced here and also only if they are visible in the scene, culling gets performed. ->
* /blender blender_geometry.cpp : sync_geometry ->
* /blender blender_mesh.cpp : sync_mesh

This is important, since I believe we have to store the previous state of geometry when it gets overwritten so we can then determine which areas of the viewport have been affected by the transformation.

## Blender abbreviations

Blender and Cycles use a myriad of abbreviations and I have been trying to uncover what they stand for:
* GHOST - General Handy Operating System Toolkit : abstracts Operating System handling
* DST = DrawState
* BKE = Blender Kernel
* BLI = Blender Internal? blender library?
* CTX = Context?
* DEG = Dependancy Graph
* CCL = Contextual C-like Language?
* OSL = Open Shading Language
* RNA = is a tribute to ribonucleic acid which forms the blueprint for the human body. These classes contain several types of pointers and are mainly used for data transfer, as far as I can tell.