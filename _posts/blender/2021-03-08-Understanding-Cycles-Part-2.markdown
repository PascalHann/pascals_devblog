---
title:  "Understanding Cycles Part 2"
date:   2021-03-08 15:20:00 +0100
categories: blender
---

## Understanding Cycles Part 2

It has been a while since my last update, so I want to give an update on my findings on how Cycles works, before writing about our project progress in another post.

### General Cycles Pipeline

There are four important steps for our projects that are handled in the following order in Cycles:

1. **Syncing the scene with blender.**  
   It is important to note, however, that only raw data like vertex positions or similar information is synced here. Data that has to be computed, like bounding boxes is updated later on.

2. **Tile generation**  
   The screenspace that needs to be rendered gets divided into tiles after the syncing with blender is finished. More on the exact process can be found further down below.

3. **Cycles scene update**
   As mentioned, data about the scene that needs to be computed rather then synced with blender is computed here. The logic flow is roughly as follows:
   * Session::run_gpu ->
   * Session::update_scene ->
   * Scene::update -> 
   * Scene::device_update -> 
   * (...)
   * GeometryManager::device_update line: 1947 object->compute_bounds
   The bounding boxes are very important for our project so i highlighted this step here.

4. **Rendering**  
   Each rendering device gets assigned one or more of the tiles that need to be rendered and renders them.

Of course, this is a very crude description and a lot more is going on in reality, but these are the steps most relevant for our project so i focus on them here.

### Tile management

The class TileManager in /render tile.h is the pivotal center point of tile management in Cycles. The sesseion (/render session.h =/= blender_session) owns the tile manager.
The session also owns RenderBuffers and DisplayBuffers.

The logic flow of tile management roughly looks as follows:

* /blender blender_session.cpp : render/bake/synchronize ->
* /render session.cpp : start ->
* /render session.cpp : run ->
* /render session.cpp : run_gpu ->
* /render tile.cpp : next ->
* /render tile.cpp : set_tiles ->
* /render tile.cpp : gen_tiles  
sliced == false => image gets split into tiles and every device gets an equal amount.  
sliced == true => image gets sliced into as many pieces as there are devices rendering it.
In everey case, the render_tile lists for each device are generated here. It also seems this step happens multiple times, perhaps for every sample?  
->
* (/render session.cpp : run_gpu/cpu ->)
* /render session.cpp : render - here a "device task" is initialized which gets a function: ->
* /render session.cpp : acquire_tile - the logic tries to get a tile via tile_manager.next_tile. If it cant get one this way, it tries to steal one from another device. ->
* /render tile.cpp - TileManager : next_tile  
In the class "TileManager" there are two vectors of lists: "denoising_tiles" and "render_tiles". For each "logical device" there is exactly 1 list of
tile indices(which are meant for the "tiles" list which holds the actual tiles) in these vectors.  
In the "next_tile" method, which gets a device as parameter, the logic basically fetches the next tile index in the list corresponding to this device or returns false, when there are none left.