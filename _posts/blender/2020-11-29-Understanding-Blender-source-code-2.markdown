---
title:  "Understanding Blender source code 2"
date:   2020-11-29 15:25:24 +0100
categories: blender
---

I have made progress in understanding Blenders transformation logic and so I want to iterate on my last post on understanding the blender source code.

When a transformation is triggered the code flow is as follows:

* wm.c : wm_event_do_handlers -> 
* transform.c : transformEvent -> 
* transform_ops.c : transform_modal -> 
* transform.c : transformApply -> 
* transform.c : viewRedrawForce

If I understand it corectly, the different transform modes like translation or rotation for example, all have their own classes like transform_translate.c and they are handeled in transform_ops.c:transform_modal during the general pipeline.  
[further reading](https://wiki.blender.org/wiki/Source/Architecture/Transform)

Sadly no one replyed to my thread on blender stack exchange yet and I am still at a loss about the exact logic of how Blender calls Cycles.