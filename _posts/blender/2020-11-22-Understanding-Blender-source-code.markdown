---
title:  "Understanding Blender source code"
date:   2020-11-22 18:03:24 +0100
categories: blender
---

In order to implement the changes into the blender and cycles source code needed to achieve the logic for my [bachelors thesis](https://pascalhann.github.io/pascals_devblog/blender/2020/11/08/Incremental-Updates-of-Path-Traced-Scenes-during-Editing.html) I am on a continuing journey to understand said source code.

Especially the code that is responsible for viewport rendering and the code for object transformation are relevant for the project.

What I could learn so far:

### Main Window Loop

* creator.c : blender main function -> 
* wm.c(WindowManager) : WM_main Main program loop -> 
* wm_draw : wm_draw_update update all non minimized windows, (swap_buffers) -> 
* wm_draw : draw_window -> 
* wm_draw : draw_window_offscreen -> 
* area.c : ED_region_do_draw draw region
* ...

### Viewport Render with Cycles

The logic for viewport rendering seems to be independant of the main window loop and gets triggered whenever a view gets updated by any sort of event. A click in a viewport suffices for example.

* draw_manager.c : DRW_draw_view ->
* draw_manager.c : DRW_draw_render_loop_ex (render Loop for external render engines?) ->
* draw_manager.c : drw_engines_init ->
* external_engine.c : external_engine_init (Cycles is an external render engine?) ->
* draw_manager.c : drw_engines_draw_scene ->
* external_engine.c : external_draw_scene ->
* external_engine.c : external_draw_scene_do ->
* Some RNA related classes I wasn't able to wrap my head around yet
  
  eventually we end up at the cycles code:

* blender_session.cpp : draw (draw call in cycles with width and height parameters. Are these the measures of the view?)

I wasn't able to identify the exact point where blender code calls cycles code. This is my main priority right now and I have started a [thread on blender stack exchange](https://blender.stackexchange.com/questions/202783/help-me-understand-the-viewport-render-pipeline-with-cycles).

### Transform Logic

The documentation on the [blender wiki](https://wiki.blender.org/wiki/Source/Architecture/Transform) is informative, but not completely up to date. The main API, described in transform.h is located in source/blender/transform not in "source/blender/include/BIF_transform.h" as far as I could tell. I wasn't able to identify the parts of the code which are responsible for updating the view after a transformation has taken place. This is another priority at the moment.

### Notes

the blender source code seems to be written in plain C for the most part, while cycles was implemented in C++.
