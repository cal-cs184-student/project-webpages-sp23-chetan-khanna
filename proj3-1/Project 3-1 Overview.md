# Project 3-1 Overview

In this project I implemented (in effect) a raytracer (optimized with path tracing).

The project at first involved implementing a fairly primitive, low-performance raytracer. I implemented the ability to generate rays (from the camera into the scene), then to raytrace a single pixel in the scene. In order to do this, we need to have the ray intersect with geometry in the scene (so we know what object and where the ray of light we’ve shot into the scene is hitting).

Our first job was therefore to implement the ability to shoot rays through the sensor at a location of our choice (since we want to trace the path of the ray through the scene such that it arrives at a point in the image of our choice).

We then need to know whether a ray intersects with objects in the scene (so that we reflect off of those objects, and eventually return!). In order to do this, we implemented intersection with two types of objects, triangles (which make up virtually everything!) and spheres.

After this, we focused on improving the performance of these intersections through a Bounding Volume Hierarchy, which allow us to have logarithmic time complexity as we check for primitives. On scenes with complex geometries, this represents a huge time saving.

After this, we started to focus on illumination of scenes. We first implemented zero-bounce lighting, which meant that we (viewing the scene through the “camera”) only saw the objects that emitted light (i.e. the area light), as we never reflected light off of any surfaces.

From there, we progressed to direct illumination, in which we bounced light off of surfaces that were receiving light (e.g. from an area light), thus enabling us to see a large part of the scenes we rendered. Among other things, this started to involve the material properties of objects and the reflection equation as we had to calculate how objects reflected light at every intersection point. The implementation of importance sampling enabled us to sample *lights* directly, thereby improving efficiency and reducing noise.

Our next job was to effectively make the direct illumination recursive, enabling us to trace rays to a chosen depth. In effect, we were now able to trace the path of light to an arbitrary depth (in terms of reflections/intersections with geometry) through the scene, therefore enabling us to depict light much more naturally in the scene and render scenes that looked more natural.

The final task of the project was to improve the efficiency of the path tracing illumination calculations. The color of certain pixels converges faster than others—often depending on the geometry of the scene and the “size” of the range of directions a path can take. We take advantage of this to implement adaptive sampling, which uses statistical calculations on a per-pixel basis to terminate early if there is a high chance that a pixel has converged to its final value.

---

**Website access:** [https://cal-cs184-student.github.io/project-webpages-sp23-chetan-khanna/proj3-1/project3Access.html](https://cal-cs184-student.github.io/project-webpages-sp23-chetan-khanna/proj2/project2Access.html)

