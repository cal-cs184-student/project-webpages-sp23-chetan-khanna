# Project 3-1 Task 5

Some parts of an image are easy to render with few samples, while others need a lot. For example, some rays shot out into a scene may take paths that are unlikely to change much every time they go there (all the BSDF's of the surfaces they hit may have reflection directions with $$\text{pdf}$$’s that are very "sharp" rather than uniform, for example).

These parts of the image will need fewer samples for the result (the `color` of the pixel) to converge.

Adaptive sampling *adaptively* changes the amount of samples-per-pixel we have to take in the image, increasing efficiency (and thus reducing noise by letting us increase our maximum samples-per-pixel rate).

The algorithm (provided in the project specification in more detail) is checking for convergence of the illuminance of the pixel; if it has, we no longer need to take more samples of this pixel. Note that since actually performing the adaptive sampling calculations has a cost, we only want to perform these calculations every `samplesPerBatch` samples.

The code for this is within `raytrace_pixel`.

After taking a sample, I get the illuminance based on the radiance from the ray that was just shot through the scene (a simple call to `radianceFromThisRay.illum()`). I add this to my running sum and sum of squares of illuminance for this pixel. I then check if the number of samples so far is divisible by `samplesPerBatch`; if it is, and this is not the first sample itself, then I perform the adaptive sampling calculations.

The check contains calculating the mean, the standard deviation, and then $$I$$. If $$I \leq  \text{maxTolerance} \times \mu$$, then I terminate early. In doing this, I set the `actualSamples` variable to the number of samples taken so far and `break` out of the `for` loop that is taking samples.

The rest of the code remains the same: I take the average of the pixel color (dividing the cumulative sum of `colorForPixel` by the number of samples taken), and I update the `sampleBuffer` for this pixel and `sampleCountBuffer`.

# Images Demonstrating Adaptive Sampling

#### Bunny

![Image.png](Project%203-1%20Task%205.assets/Image.png)

![Image.png](Project%203-1%20Task%205.assets/Image%20(2).png)

#### Spheres

![Part5SpheresRender.png](Project%203-1%20Task%205.assets/Part5SpheresRender.png)

![Part5SpheresRender_rate.png](Project%203-1%20Task%205.assets/Part5SpheresRender_rate.png)

Like in task 4, the artifacts near the bottom of the sphere exist due to a max ray depth that is too low in this render. Inputting 6 or greater to the `-m` argument when running the program prevents this.

