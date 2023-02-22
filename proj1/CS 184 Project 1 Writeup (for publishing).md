# CS 184 Project 1 Writeup (for publishing)

# Overview

I implemented triangle rasterization in this CS 184 project. In other words, I took a virtual drawing (e.g. a cube or a logo) and converted that drawing (that “scene”) into a set of pixel values that I could place in the frame buffer so that they are drawn onto our display.

At first, this was just basic triangle rasterization, which would check whether for each pixel in the frame buffer, a triangle (that we wanted to draw) would be there, and then in that case we would place the appropriate value (a `color`—at first no textures, just a passed in `Color`) for that pixel in the frame buffer.

I then implemented progressively more advanced methods and techniques to perform rasterization. The first improvement I made was supersampling, in which we took multiple samples for each pixel in the frame buffer (so we got multiple colors) and then averaged them out for to get our final `Color` value which we placed for that pixel in the frame buffer. Since this was meant we were sampling the image at a higher frequency, this reduced aliasing.

Subsequently, we implemented the ability to transform, scale, and rotate images. These were done through using the appropriate matrices (e.g. a rotation matrix for rotation).

In order to do implement texturing properly, I had to implement Barycentric coordinates—effectively a coordinate system defined by the three vertices of the triangle, each given a “weight”. This allowed us to perform interpolation properly (hence why an interpolated color triangle was possible), in addition to performing the correct conversion between screen and texture space (since Barycentric coordinates, in a sense, give us our position in the triangle relative to the triangle’s vertices itself).

After this, I implemented texturing, so that rather than filling with triangles with just a solid color, we could sample the `Color` from a `Texture` object for each specific sample we were taking during the rasterization process. This meant that I had to be able to convert to texture space from screen space correctly (i.e. perform the correct transformation) so that we could get the right `Color` for where we were in the texture. There were multiple ways to perform the actual sampling, which I elaborate on below.

Finally, I implemented sampling from different mipmap levels. We want to reduce aliasing by taking a lower-resolution mipmap (i.e. higher level mipmap) for far away objects (since each sample will be much “farther away” when placed in texture space for minified (or far away) textures), and vice versa for closer objects. To do this, as we’re rasterizing, we check the distance between this sample and an adjacent sample (the specific adjacent sample will be elaborated upon below), and, based on that, choose a mipmap level that we will sample from. Furthermore, we also implemented a technique to use *two levels* of the mipmap, since moving between each mip level could otherwise look jarring. In this case, for each screen sample we take, we actually take *two* (rather than one) texture space samples from two adjacent mipmap levels (derived from our normal mipmap level calculation). We then perform bilinear interpolation between the two to get our final texture space sample. This "combination" of two texture space samples roughly approximates continuous mipmap levels.

Share thoughts on itneresting thigns from the project

# Task 1

- Walk through how you rasterize triangles in your own words.
- Explain how your algorithm is no worse than one that checks each sample within the bounding box of the triangle.
- Show a *png* screenshot of *basic/test4.svg* with the default viewing parameters and with the pixel inspector centered on an interesting part of the scene.

### Basic Idea of the Algorithm

The algorithm is based on the idea of checking if the point (pixel) we are rasterizing falls inside the triangle or not. We can do this by recognizing that a triangle is the intersection of three half-planes—each half-plane split from a “normal/full” plane by the edge of a triangle (which acts as a line in space—we assume it extends to infinity). If we can check that the point falls on the correct side of *all three* half planes (whose intersection defines the triangle we are trying to rasterize), then we know that the point (pixel) we are rasterizing is *within* the triangle, and we must thus fill `color` in the frame buffer.

### Algorithm Walk Through

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

### Attached Picture of `test4.svg`

# Task 2

- Walk through your supersampling algorithm and data structures. Why is supersampling useful? What modifications did you make to the rasterization pipeline in the process? Explain how you used supersampling to antialias your triangles.
- Show *png* screenshots of *basic/test4.svg* with the default viewing parameters and sample rates 1, 4, and 16 to compare them side-by-side. Position the pixel inspector over an area that showcases the effect dramatically; for example, a very skinny triangle corner. Explain why these results are observed.
- *Extra credit:* If you implemented alternative antialiasing methods, describe them and include comparison pictures demonstrating the difference between your method and grid-based supersampling.

## What Supersampling Is

Supersampling means we will sample inside multiple places within the pixel and average those values together. By sampling multiple times per pixel, we increase the *sampling frequency*. An increased sampling frequency means that there we are not sampling with an excessively low frequency in parts of the image that encode high frequency/fast-changing signals (e.g. the edge of a triangle, where the signal can suddenly change from, say, white to the color of the triangle). This decreases aliasing, which can be seen as jaggies in the image.

## Supersampling algorithm walk through

#### Overview

In order to supersample, we modify a couple of functions. Firstly, the algorithm in `rasterize_triangle` is modified. We now sample *multiple* times per pixel and put this in a `sample_buffer` rather than directly drawing to the screen by calling `draw_pixel`. Later, once we have finished `rasterize_triangle`, we will go through the `sample_buffer`, and average the `Color` samples inside `sample_buffer` that correspond to every pixel together. This gives us our final `Color` for every pixel. To draw these, we put them in the frame buffer, inside an array called `rgb_framebuffer`. (Note that we first scale our `Color` from `float` values in the range 0-1 to `ints` in the range 0-255)

#### Data structure: `sample_buffer` indexing

In `rasterize_triangle`, we place our sampled values in a 1D vector called `sample_buffer` of length `width * height * sample_rate`. `sample_buffer` contains our samples in the form of `Color` objects. In the case of `sample_rate = 1`, sample $$(x,y)$$ is stored at index `width * y + x`. When `sample_rate = 1`, there is only one element per pixel in the `sample_buffer`.

More generally, sample $$samRe$$ of pixel $$(x,y)$$ is given by `sample_buffer[(curr_y * sample_rate * width) + (curr_x * sample_rate) + samRe]`.

The first two terms, in parentheses, represent jumping to pixel $$(x,y)$$. The last term, `s`, represents a sample for that pixel. We keep all the samples for a pixel together in the `sample_buffer`.

**Bonus explanation:** The first term is `curr_y * sample_rate * width` because `y` and `width` are both "stretched" by `sqrt(sample_rate)`, leading to a `sample_rate` multiplier. `x` must be multiplied by `sample_rate` because there are `sample_rate` samples for every pixel, so we must skip `sample_rate` elements when going to the next pixel.

### Overview of Modifications in `rasterize_triangle`

Assume the setup is same as in task 1: we are given the vertices of the triangle $$(x_i, y_i)$$ (in screen-space) and a `color` we must fill into the triangle. Furthermore, many aspects remain the same: we find a bounding box (in the same way), and the logic/mathematics behind choosing whether a ***sample***  (not ”pixel”) is in the triangle remains the same.

In order to take multiple samples per pixel, we add two *extra* `for` loops inside the existing double-`for` loop. These extra for loops allow us to iterate in the $$x$$ and $$y$$ dimensions, taking a sample at each iteration, ***inside*** a single pixel. This means we have the following structure:

```cpp
// Pre-processing: finding the bounding box, creating the normal vectors, etc.
// First set of for loops for a pixel
for (int curr_x = start_X; curr_x <= end_X; curr_x++) {
    for (int curr_y = start_Y; curr_y <= end_Y; curr_y++) {
      // Inside a pixel
      unsigned long sampleIndex = 0;
      float startSamplePoint = 0.5 * (float(1)/float(sqrt(sample_rate)));
      float incrementSampleBy = float(1)/float((sqrt(sample_rate)));
      unsigned long startSampleBufferIndex = jumpToPixelX + jumpToPixelY;
      // Some setup related to jumping to the correct pixel inside the sampleBuffer
      // Second set of loops that go through samples *in* a pixel
      for (int x_sample_iteration = 0; x_sample_iteration < sqrt(sample_rate); x_sample_iteration++) {
                  for (int y_sample_iteration = 0; y_sample_iteration < sqrt(sample_rate); y_sample_iteration++) {
                  // Now inside a sample for this pixel
                  // We perform the half-plane intersection test here and update the sampleBuffer at the corresponding index if necessary
```

- **Inner for loop start and end:** For each pixel, we want to take `sample_rate` samples. This means we will iterate from 0 to `sqrt(sample_rate)` in the $$x$$ and $$y$$ dimensions.
- `incrementSampleBy`**:** For each sample, we iterate the location of our sample point by adding $$\frac{1}{\sqrt{\text{sample rate}}}$$. This iteration happens separately in both $$x$$ and $$y$$. This follows directly from the point above: if we are be taking $$\sqrt{\text{sample rate}}$$ samples in each dimension, we must add $$\frac{1}{\sqrt{\text{sample rate}}}$$ to cover the entire "sample area" in each dimension for the pixel.
- `startSamplePoint:` The first sample point for a pixel will be given by $$\frac{1}{2{\sqrt{\text{sample rate}}}}$$ in each of $$x$$ and $$y$$, relative to the pixel. This means the first sample point in a pixel will be given by $$(x + \frac{1}{2{\sqrt{\text{sample rate}}}}, y + \frac{1}{2{\sqrt{\text{sample rate}}}})$$. This makes intuitive sense: the larger the sample rate, the “earlier” in the pixel we need to start sampling, so the first sample happens at an earlier point in time. The $$1/2$$ occurs because the "tightening" of spacing between samples as `sample_rate` increases happens on both "ends".
- `sampleIndex`**:** this is `samRe`, used to place a sample in the correct spot in `sample_buffer`. It starts at 0 for every pixel and is incremented for every sample we take in the pixel.

**Based on these points and the parameters as shown in the code block above, we can therefore find the location of the sample point in the following manner:**

- `float xSamplePoint = curr_x + startSamplePoint + (x_sample_iteration * incrementSampleBy);`
- `float ySamplePoint = curr_y + startSamplePoint + (y_sample_iteration * incrementSampleBy);`

**The test remains the same as in Task 1: we take the same dot products and test if the sample is in the triangle.** If so, we place it in the appropriate index in `sample_buffer`.

### Demonstration

## Demonstration

- [ ] TODO: Place the demonstrated images here.

# Task 3

## How this works

We use linear algebra to apply transformations to images, including `rotate`, `scale`, and `translate`.

In order to be able to use matrices for (affine) transformations, we use homogeneous coordinates, which involves appending an extra *row* to the bottom of the matrix and setting (in this situation) the bottommost element of the last column vector to 1. Using matrices helps speed up calculations.

# Demonstration

- [ ] TODO: Place the demonstrated images here.

# Task 4

#### High-level intuition

Barycentric coordinates are a coordinate system based on specific “weights” connected to each of the three vertices of our triangle. We can use different sets of weights to uniquely describe a point in space, generally on the triangle. If a vertex’s has a higher “weight” associated with it, the point is going to be closer from that vertex.

- [ ] Place image to aid me in explanation of Barycentric coordinates

#### Definition

> More specifically, a point can be described by the following

> $$(x,y) = \alpha A + \beta B + \gamma C$$

> where  $$\alpha, \beta, \gamma$$ are weights corresponding to vertices $$A,B,C$$ of the triangle respectively.

Note that this means that, by definition, $$\alpha, \beta, \gamma$$ range from 0 to 1.

This structure means that we can easily define a property at the three vertices of a triangle, and then perform linear interpolation to get the “state” in all the other parts of the triangle (after all, it is just a matter of scaling the weights).

#### Useful property

Barycentric coordinates have the following useful property:

> If $$\alpha + \beta + \gamma = 1$$ and all three weights are non-negative, then the point $$(x,y)$$ is inside the triangle.

This means we can convert a point during rasterization to barycentric coordinates (i.e. find $$\alpha, \beta, \gamma$$) and using the rule above quickly check if the point is contained within the triangle or not.

This means we no longer need to perform the half-plane intersection tests we did in previous tasks inside `rasterize_triangle`. Instead, after converting to barycentric coordinates, we just check if this property is fulfilled.

### In Code

In order to perform interpolation of color, I first converted each pixel into barycentric coordinates.

For a sample $$\text{p}$$, the conversion for $$\text{\alpha}$$ was performed by taking the cross product of vector $$\text{BC}$$ and vector $$\text{p - A}$$. The conversion was analogous for $$\text{\beta}$$. In order to minimize issues with floating-point error, $$\gamma$$ was defined as $$1 - \alpha - \beta$$. The usual half-plane line intersection test was nevertheless performed; it was done before computing barycentric coordinates and did not appear to reduce performance significantly.

After converting into barycentric coordinates, I calculated the color of the sample in the standard format $$\alpha A_\text{color} + \beta B_\text{color} + \gamma C_\text{color}$$, where $$A_\text{color}, B_\text{color}, C_\text{color}$$ are the colors of each of the triangle's vertices respectively. I put the result into the `sample_buffer` at the appropriate index (same as Task 2).

## Demonstration

- [ ] Place demo here.

# Task 5

In order to draw textures, we need to also sample texture space. In particular, for a screen space sample we want to draw to the frame buffer, we now need to sample the *texture* at the corresponding point on that object.

More specifically, for every screen space sample we take, we must:

1. Convert into texture space coordinates (from $$x,y$$ into $$u,v$$)
2. Sample the texture (getting a `Color` object in the case of this project)
3. Put this in the sample buffer.

The two ways of sampling a texture (given a screen space sample) are known as nearest-pixel sampling and bilinear interpolation. I elaborate on these in the sections below.

## My Implementation

### Basic Code Idea

For a point I want to place in the frame buffer, I (mostly) convert to texture space in `rasterize_textured_triangle`. I then call the associated `Texture` object's sampling method (either `nearest` or `bilinear` based on the user's setting) with that point. The `Texture`'s `sample` method then returns a `Color` object, which I place in the `sample_buffer` at the appropriate index (see Task 2 for more detail on indexing).

### Elaboration: Conversion into texture space

I converted into texture space in `rasterize_textured_triangle`. In order to do this, I first converted into barycentric coordinates in identical fashion to task 4 to get `alpha`, `beta`, and `gamma`. (Intuitively, these gave me the position of a point "relative to the vertices of the triangle," which is extremely useful as this enables me to directly go to the corresponding position in the triangle's texture space.)

More specifically, I used $$\alpha$$, $$\beta$$, and $$\gamma$$ to get the corresponding point in texture space by multiplying these weights with the same vertices of the triangle, except given in texture space. These were already passed in to `rasturize_triangle` as `u0` (corresponding to vertex $$x_0$$), `v0` (corresponding to vertex $$y_0$$), `u1`, and so on.

This allowed me to get the relative position of the point in texture space. To get the actual point $$(u,v)$$, I needed to scale up the points by the `length` and `width` of the texture respectively.

Since the `length` and `width` could change based on the mipmap's level (elaborated upon in question 6), I elected to pass an unscaled vector to the `Texture` object's `sample_bilinear` or `sample_nearest` (based on the user's settings) options. The `sample_bilinear` and `sample_nearest` scale by the mipmap's level. (Note that in this question I could have scaled up by `tex.width`/`tex.length` immediately but I would have had to change this for Task 6.)

As noted above, the method in `Texture` then returns a `Color` object to `rasterize_textured_triangle`, which I place in the `sample_buffer` at the appropriate index (see Task 2 for more detail on indexing).

### Elaboration: Sampling in texture space

There are two ways to sample in texture space. I elaborate on the code here, and provide some theory in the section below.

#### Bilinear sampling (`Texture::sample_bilinear`)

In bilinear sampling, I take the four closest points to the sample point in texture space. I sample these points individually, and then I perform linear interpolation to get the final sample `Color` that is returned to `rasterize_textured_triangle`.

**In more detail:**

I achieve this by first scaling the `uv` vector by `mip.width` and `mip.height` (as detailed above). This provides us with the coordinates of the sample point in texture space.

We get the nearest four points in texture space by performing all combinations of `floor`ing and `ceil`ing $$\text{u}$$ and $$\text{v}$$. I get the `texel`s for each of these four points and store them.

I then perform linear interpolation based on the original sample point’s coordinates. I first perform a linear interpretation between the two "lower" texture points (p`C` and p`B)`. I then perform a linear interpolation between the two “upper” texture points (in the code p`A` and p`D)`. The results of these two interpolations give us one C`olor` each, so I then combine the results from these two "helper” interpolations with one final linear interpolation. This gives us one final `Color`.

Each linear interpolation is performed in the following manner:

- Create a “line” between the two points we are interpolating between. This line travels in the $$\text{+x}$$ direction for the helper interpolations, and the $$\text{+y}$$ direction for the vertical interpolation.
- I decide the proportion based on the original sample point (from the scaled up `uv` vector). For the helper interpolations, a larger `u` component implies a higher weight on from the points on the right (the `v` component is not used since the lines are flat). For the final interpretation, a larger `v` component implies a higher weight on result of the upper linear interpolation (the `u` component is not used since the line we create is purely vertical).
- I then calculate the color by multiplying the $$\text{interpolated color = (proportion) \times (end point color) + (1 - proportion) \times (start point color)}$$.
   - where, for the helper interpolation: $$\text{start point}$$ is the left texture point and  $$\text{end point}$$ is the right texture point
   - for the last linear interpolation, $$\text{start point}$$ is the result of the lower helper interpolation and $$\text{end point}$$ is the result of the upper linear interpolation

I get the texel at this coordinate by calling `mip.get_texel(x, y)` and returning the `Color` object that it returns.

#### Nearest-pixel sampling (`Texture::sample_nearest`)

Nearest pixel sampling is simpler than bilinear sampling; it simply takes the nearest texture point to our sample point in texture space.

Upon receiving a `uv` vector, I scale it by `mip.width` and `mip.height` (as detailed above). This provides us with the coordinates of the sample point in texture space.

I then round the `u` and `v` components (now called `x` and `y`) of this scaled sample point, which will provide us with the nearest texture sample point.

I get the texel at this coordinate by by calling `mip.get_texel(x, y)` and returning the `Color` object that it returns.

Add rounding

May need to change scaling for `mipmap`

### A note on bilinear vs nearest-pixel sampling

Because we are *sampling* texture space, all the usual caveats of sampling apply. Most importantly, this includes aliasing. Aliasing is most clear in the cases of texture magnification or, especially, texture minification.

Bilinear interpolation, as described above, is in effect an antialiasing technique. While the nearest pixel technique is only taking a single sample of the texture for each screen space sample, the bilinear interpolation strategy takes the *four* closest texture points and interpolates between them to return a single value. This means we sample at a higher frequency at a higher frequency and thus acts as an antialiasing technique.

### Demonstration of differences

- [ ] Comment on the relative differences. Discuss when there will be a large difference between the two methods and why.

# Task 6

When we are minifying textures—or drawing textures that are far away—one screen space sample corresponds to a large portion of texture space. This means that as we continue to take screen space samples and put them into texture space, we find that they are spaced very far apart (and therefore the texture samples we take are spaced very far apart). This, in turn, means that we are sampling texture space at a very low rate.

As noted in the section above, this can cause *aliasing*: our sampling frequency is much lower than the frequency of the signals of the texture space.

At the same time, we also want to *keep* higher-resolution textures, since if we want to draw a larger version of the image/the object we are drawing comes closer in our scene, then we would face a worse result (and have to perform interpolation in order to maintain an acceptable result).

We thus use *mipmaps*. Mipmaps store a texture at different resolutions, called *levels*.

For example, we store a (very) high-resolution texture at Level 0, e.g. 1024x1024. As we down one level, we halve the resolution in each dimension—going all the way down to 1 pixel at the lowest level! Mipmaps are relatively space-efficient: it turns out it doesn’t take a lot more space than just the original high-resolution texture image (one third more memory).

Depending on what our sampling of texture space is, we can choose an appropriate *level* of mipmap to reduce aliasing. (Intuitively, we want to choose the mipmap that has texture points aligning most closely with our sampling frequency).

We choose a level by looking measuring the distance between screen space samples for every screen space sample we take. I elaborate in [explanation of get_level](craftdocs://open?blockId=C906A990-6CF3-4AFB-BBB4-C1C1154D027C&spaceId=067f573b-b0da-d02e-e3ba-486aa57dc31a) below.

Note that we can also choose to use a technique to use *two levels* of the mipmap, since moving between each mip level could otherwise look jarring. In this case, for each screen sample we take, we actually take *two* (rather than one) texture space samples from two adjacent mipmap levels (derived from our normal mipmap level calculation). We then perform bilinear interpolation between the two to get our final texture space sample. This "combination" of two texture space samples roughly approximates continuous mipmap levels.

## Coding

For every screen space sample $$(x,y)$$ in `rasterize_triangle`, we find the barycentric coordinates in order to perform texture sampling (as usual). However, now, we also find the barycentric coordinates for $$(x+1, y)$$ and $$(x, y+1)$$ (in the same way). We construct (unscaled) vectors for each of these points, and we place them in a `SampleParams` struct.

We will use these extra barycentric coordinates soon, in T`exture,` in order to determine the distance between sample points in texture space, and thus decide on a mipmap level to use when sampling ($$(x,y)$$

We pass the `SampleParams` struct to `Texture::sample`. `Texture::sample` takes the pixel-sampling and level sampling settings (`psm` and `lsm`) into account. Unless `lsm` is set to `L_ZERO`, sample will call `get_level`, which returns the mipmap level (as a `float`) to perform sampling at.

#### `Texture::sample`: `lsm` set to `L_NEAREST`

If `lsm` is set to `L_SAMPLE`, then we round the mipmap level to the nearest integer and call the appropriate sampling method (`sample_bilinear` or `sample_nearest`).

#### `Texture::sample`: `lsm` set to `L_LINEAR`

If `lsm` is set to `L_LINEAR`, then we perform interpolation between the samples from two adjacent mipmap levels as described above.

In particular, to get a final sample, we:

1. Calculate `get_level` as normal. Note `get_level` returns a `float`.
2. We take the `ceil` and `floor` of the `level` in order to get two adjacent mipmap levels.
3. We take the sample from each of the two mipmap levels, returning a `Color`.
4. We perform linear interpolation based on the value of `get_level`. We use the fractional part of the `float` (i.e. remove the integer) to determine the weights between the two samples.
   - As an example: Say the original `level` was `2.9`. In this case, the fractional part is `0.9`, and thus the higher mipmap level will be weighted heavily, and the sample `Color` will be strongly weighted towards it.
1. We return the interpolated `Color`.

Note how, just like when performing linear interpolation in task 5, if `level` is an integer, the interpolation still occurs, except with a weight value of 0. This doesn't matter, since the "counterweight" is 1, and multiplies (by definition) with the same value.

Furthermore, note the stricter bounds checks performed here compared with the `L_LINEAR` case; this is necessary because `ceil(level)` must not exceed the highest mipmap level.

#### Explanation of `get_level`

`get_level` works by taking the $$\text{uv}$$ vectors corresponding to the $$(x+1, y)$$ and $$(x, y+1)$$ (and, as usual, scaling them up by `width` and `height`). Call these vectors $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$ respectively.

**Then:**

1. For each of $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$: Create a vector representing the distance between the original sample point in texture space (call this point $$(u,v)$$) and $$\text{uv_{x+1}}$$ (or $$\text{uv_{y+1}}$$ respectively).
   - Call the corresponding vectors $$\frac{d(u,v)}{dx}$$ and $$\frac{d(u,v)}{dy}$$. Note how $$\frac{d(u,v)}{dx}$$ is a vector showing how much $$(u,v)$$ change in texture space for a change of $$x$$ in screen space. (This is analogously true for $$\frac{d(u,v)}{dy}$$)
1. For each of $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$: Use Pythagoras’ Theorem to find the length of each vector.
   - We split the vector $$\frac{d(u,v)}{dx}$$ into it's $$u$$ and $$v$$ components respectively, which are known. These form the legs of a triangle. The hypothesis of this triangle is given by Pythagoras' Theorem, and represents the length of $$\frac{d(u,v)}{dx}$$. (This is analogously true for $$\frac{d(u,v)}{dy}$$)
1. By convention, we take the maximum length between $$\frac{d(u,v)}{dx}$$ and $$\frac{d(u,v)}{dy}$$ to "represent" the distance between samples in texture space.
2. Our level of the mipmap will be the $$\text{log_2}$$ of this length; this is because each higher level of the mipmap is half the resolution, so doubling the length between screen space samples should lead to an increase of *one* mipmap level.

## Tradeoffs between speed, memory usage, and antialiasing power

| **Pixel sampling method** | **Level sampling method** | **Speed and memory usage (performance)** | **Antialiasing** |
| ------------------------- | ------------------------- | ---------------------------------------- | ---------------- |
| Nearest Pixel             | Zero                      | Best                                     | Worst            |
| Nearest Pixel             | Nearest                   | Good                                     | OK               |
| Nearest Pixel             | Linear                    | OK                                       | Good             |
| Bilinear Interpolation    | Zero                      | OK                                       | OK               |
| Bilinear Interpolation    | Nearest                   | Bad                                      | Good             |
| Bilinear Interpolation    | Linear                    | Worst                                    | Best             |

**Brief Comment:**

The fastest method is to use `L_ZERO` and `P_NEAREST`. This requires no calls to `get_level`, no interpolations performed anywhere, and no need to calculate sample points in texture space for adjacent points. However, it leads to significant aliasing, both of Moiré and Jaggies.

`L_NEAREST` and especially `L_LINEAR` will improve this but entail calls to `get_level`, the calculation of more vectors inside `rasterize_textured_triangle` and `get_level`. `L_LINEAR` will also lead to an interpolation operation inside `sample` itself, further increasing performance impact.

The best, of course, is `P_LINEAR` and `L_LINEAR` which will lead to reduced aliasing from sampling the texture (by enabling increased sampling frequency when sampling the texture), and also the choice of the correct mipmap (by enabling the texture to be of the best possible mipmap level, and therefore "frequency”)

