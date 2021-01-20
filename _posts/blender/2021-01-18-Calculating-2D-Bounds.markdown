---
title:  "Calculating 2D Bounds"
date:   2021-01-18 15:20:00 +0100
categories: blender
---

Since the last post we have implemented our first code changes. I want to go over them in this one:

## Storing the bounds from the previous syncing step

We want to achieve a renderer that only rerenders affected parts of the image when changes occur within the scene. Our first approach to that, is to rerender the new and old 2D pixel space bounding boxes of an object, when it gets transformed.  
For this, we need the bounding box of the object previous of the current syncing step in addition to the current one.  
We added a field to **/render/object.h**:

{% highlight cpp linenos %}
{% raw %}
BoundBox old_bounds;
{% endraw %}
{% endhighlight %}

We populate this field in the method **/blender/blender_session.cpp#synchronize**
before the objects get synced. We simply replace the old_bounds with the current bounds, which in turn will be overwritten with the future bounds during the following syncing step.

{% highlight cpp linenos %}
{% raw %}
/* Update old bounds before overwriting new bounds */
for (Object *object : scene->objects) {
  object->old_bounds = object->bounds;
}
{% endraw %}
{% endhighlight %}

## Calculating 2D Bounds

The bounds we store in **/render/object.h** are 3D world space bounding boxes. In order to update the corresponding parts of the screen, we need to calculate the 2D pixel space bounding boxes from them.  
To do so, we added a method to the object class:

{% highlight cpp linenos %}
{% raw %}
/* Compute 2D raster space bounding box from 3D world space Bounding Box */
BoundBox2D Object::compute_raster_bounds(BoundBox bbox, ProjectionTransform worldtoraster)
{
  BoundBox2D result = BoundBox2D();
  vector<float3> bb_corners = vector<float3>({{bbox.min.x, bbox.min.y, bbox.min.z},
                                              {bbox.min.x, bbox.max.y, bbox.min.z},
                                              {bbox.min.x, bbox.min.y, bbox.max.z},
                                              {bbox.min.x, bbox.max.y, bbox.max.z},
                                              {bbox.max.x, bbox.min.y, bbox.min.z},
                                              {bbox.max.x, bbox.max.y, bbox.min.z},
                                              {bbox.max.x, bbox.min.y, bbox.max.z},
                                              {bbox.max.x, bbox.max.y, bbox.max.z}});

  for (int a = 0; a < bb_corners.size(); a++) {
    bb_corners[a] = transform_perspective(&(worldtoraster), bb_corners[a]);
  }

  float3 max = {-INFINITY, -INFINITY, -INFINITY};
  float3 min = {INFINITY, INFINITY, INFINITY};

  for (int a = 0; a < bb_corners.size(); a++) {
    min.x = std::min(min.x, bb_corners[a].x);
    min.y = std::min(min.y, bb_corners[a].y);
    min.z = std::min(min.z, bb_corners[a].z);

    max.x = std::max(max.x, bb_corners[a].x);
    max.y = std::max(max.y, bb_corners[a].y);
    max.z = std::max(max.z, bb_corners[a].z);
  }

  result.left = min.x;
  result.right = max.x;
  result.top = min.y;
  result.bottom = max.y;

  return result;
}
{% endraw %}
{% endhighlight %}

This simply extracts the corners of the 3D bounding box and makes use of the already implemented method "transform_perspective" to calculate a 2D screen space bounding box.

## Next steps

Looking forward, we will try to feed the information we prepared into the rendering logic, a colleague of us is currently modifieng, so it can render only parts of the image rather then the whole.

Follow the development on our [git repository](https://github.com/PascalHann/BA_WS2020/tree/feature/store_previous_geometry).