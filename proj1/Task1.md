# Task 1

## Basic Idea of the Algorithm (Walking Through Rasterization of Rasterizing Triangles)

The algorithm is based on the idea of checking if the point (pixel) we are rasterizing falls inside the triangle or not. We can do this by recognizing that a triangle is the intersection of three half-planes—each half-plane split from a “normal/full” plane by the edge of a triangle (which acts as a line in space—we assume it extends to infinity). If we can check that the point falls on the correct side of *all three* half planes (whose intersection defines the triangle we are trying to rasterize), then we know that the point (pixel) we are rasterizing is *within* the triangle, and we must thus fill `color` in the frame buffer.

## Algorithm Walk Through

We are given the task to rasterize a triangle. We are given the vertices of the triangle $$(x_i, y_i)$$ (in screen-space) and a `color` we must fill into the triangle.

Before starting, note that we want to create a bounding box around the triangle so that we don’t needlessly iterate through parts of the frame buffer and perform tests to see whether we should draw the triangle that *definitely* will not have the triangle. This is the first thing we will do.

1. **Find and create the bounding box:** Create a list of the $$x$$ and $$y$$ points of the triangle, and find the minimum $$x$$, maximum $$x$$, minimum $$y$$, and maximum $$y$$ points of the triangle (all independent of one another—each min/max point in each dimension can belong to a different vertex—if we didn't do this, then we would make our bounding box too small). **We use these min/max points below in our `for` loops to "simulate" a bounding box.**
2. **Quick error/sanity check:** we check the vertices of the triangle fit in the frame buffer, i.e. we take all four min/max points we found and check they're more than or equal to `0` and are less than the `height` (for $$y$$) or `width` (for $$x$$). If they're not, we return out of the function.
3. **We create three vectors that are normal to the edges of our triangle.** Specifically, we first create three vectors for each edge of the triangle. We then get the normals of these vectors  by swapping the $$x$$ and $$y$$ components of the vectors and negating the (new) $$x$$ component.
4. **We now start iterating through every pixel of the frame buffer within the bounding box.** We perform the [Basic Idea of the Algorithm](craftdocs://open?blockId=29B500BF-E8DE-496C-B2AC-9F9C7E2CF263&spaceId=067f573b-b0da-d02e-e3ba-486aa57dc31a) above.
   1. This means that we start at pixel (min $$x$$, min $$y$$), going up to pixel (max $$x$$, max $$y$$), pixel by pixel.
   2. For every edge:
      1. We create a vector from each vertex to the current pixel; we call this the “pixel vector”. This is from the “first” (i.e. leftwards in typical illustrations) vector of that line to the current pixel for all edges of the triangle in order to maintain consistency. I elaborate on the need for this vector below.
   3. Since the dot product tells us the side of the half-plane we are on (by telling whether the current pixel is acute/obtuse to the normal, which is 90º to the line itself), we take the dot product of the normal with the “pixel vector”. If the dot product is more than 1, we know that the pixel lies on one side of the half-plane; if less than 0, then the other, and that it lies on the line itself if the pixel is equal to 0.
   4. We check if the dot product is simultaneously more than or equal to 0 for every edge of the triangle; if so, it lies in the intersection of the three half-planes that define the triangle, and thus should be rasterized. We thus call `draw_pixel(color)` to draw this pixel.
      1. Note that this would also be true if the dot product for all three edges is simultaneously equal to less than 0; they would also all lie in the same half-plane then and be in the triangle.
      2. The “equal to” in “equal to 0” is due to edge-drawing rules.

### Why This Algorithm is No Worse Than One That Checks Each Sample Within the Bounding Box

My algorithm *creates* a bounding box around each triangle it has to rasterize that is as "small" as possible (since I create a list of the $$x$$ and $$y$$ points of the triangle, and find the minimum $$x$$, maximum $$x$$, minimum $$y$$, and maximum $$y$$ points of the triangle), so it is not checking any more samples than just those around the bounding box.

## Attached Picture of `test4.svg`

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/781E0B2D-7CD5-42E1-8684-6E1B074183E1/CB84B1DA-8D40-41B8-9E1C-77AEBFD0D9D7_2/unguLQpAi2qRgxk2zmeoZnzCMG5yp5wtg1iL9gfa5nwz/Image.png)

