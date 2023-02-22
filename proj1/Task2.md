# Task 2

## What Supersampling Is

Supersampling means we will sample inside multiple places within the pixel and average those values together. By sampling multiple times per pixel, we increase the *sampling frequency*. An increased sampling frequency means that there we are not sampling with an excessively low frequency in parts of the image that encode high frequency/fast-changing signals (e.g. the edge of a triangle, where the signal can suddenly change from, say, white to the color of the triangle). This decreases aliasing, which can be seen as jaggies in the image.

# Supersampling algorithm walk through

### Overview

In order to supersample, we modify a couple of functions. Firstly, the algorithm in `rasterize_triangle` is modified. We now sample *multiple* times per pixel and put this in a `sample_buffer` rather than directly drawing to the screen by calling `draw_pixel`. Later, once we have finished `rasterize_triangle`, we will go through the `sample_buffer`, and average the `Color` samples inside `sample_buffer` that correspond to every pixel together. This gives us our final `Color` for every pixel. To draw these, we put them in the frame buffer, inside an array called `rgb_framebuffer`. (Note that we first scale our `Color` from `float` values in the range 0-1 to `ints` in the range 0-255)

### Data structure: `sample_buffer` indexing

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

# Demonstration

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/CDD8B723-75BE-4008-BCBD-6CFFB0E31E51/9C3F586F-C6DA-4C68-AFFF-BFC41A00605D_2/TlJuuyFIHXwjJXby4JCHjTxJiqiuLBOXF6UWbyY8Y9oz/Image.png)

The following image shows supersampling at `sample_rate` 4.

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/CDD8B723-75BE-4008-BCBD-6CFFB0E31E51/6224C37A-2DC7-41FC-A13E-B67515146BA1_2/ZFVu7xKBUaFsgYiuIk0tzC4qTJhpxRmAIiyzKFoINwIz/Image.png)

The following image shows supersampling at `sample_rate` 9.

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/CDD8B723-75BE-4008-BCBD-6CFFB0E31E51/5B09CFC7-52F2-4CFA-AA1E-B6B09486D5DC_2/b23GcRyjV4I4gXx8sxRQFIRkHoPTfToUO9TdOyzCYwkz/Image.png)

The following image shows supersampling at `sample_rate` 16.

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/CDD8B723-75BE-4008-BCBD-6CFFB0E31E51/18236C2F-C64F-4888-8B4F-C37D3631987B_2/xnf6GdSqZIqEbaq09wLTaVjQRK3e7qjjxRksvYm6zaYz/Image.png)

### Why these results are observed

As `sample_rate` increases, the edges of the triangle become softer. This follows from my explanation above, so I will explain it briefly here. The reason for this is detailed above. At pixels near the edges of a triangle, some samples may pass the "line test" and therefore be considered within the triangle, while some others may not. Since the final color in every screen pixel is an average of these samples, this means the colors in those pixels will be more faded.

