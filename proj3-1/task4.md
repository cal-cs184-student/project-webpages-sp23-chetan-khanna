# Project 3-1 Task 4

# Implementation of the Indirect Lighting Function

The indirect lighting function is implemented through `at_least_one_bounce_radiance`, which recursively extends `one_bounce_radiance` to implement global illumination.

The function is called after having shot a ray through the scene and having found it’s intersection point with the surface. We therefore already know the intersection and ray details.

We first thus get the light on the point from other sources being reflected back (i.e. `one_bounce_radiance`). This is achieved through `L_out += one_bounce_radiance(r, isect)`.

We then proceed to the recursion.

We first need to setup the next bounce. In order to do this, we need to create a ray that we shoot out from this point to the next point (so we can find the light coming from the point in that ray’s direction). Because this depends on the way the surface's material reflects light, we have to use `hit_p`’s object’s BSDF function which gives us the a direction vector for the ray of "outgoing" (physically incoming) light along with a $$\text{pdf}$$. We thus call `isect.bsdf->sample_f(w_out, &w_i_object, &previousRayDirPDF)` which gives us this direction for the next bounce (given the direction `w_out` that we hit this point; i.e. the camera (at first)/ray from previous intersection point (recursion)). It also gives us the `reflectance` of the current object (the necessary term for our reflection equation that we used in Task 3 too). We then cast the ray to the next object in a similar way to before (setting `min_t` to  `EPS_F`, converting `w_i_object` to get the ray's direction vector `w_i_world` by multiplying by `o2w`, set it's origin at `hit_p` etc).

The important thing to note here is that we reduce this new ray’s `depth = r.depth - 1`, i.e, it's depth drops by one compared to the previous ray that hit this point. We keep track of depth so that we don't go above `max_ray_depth` bounces.

Like with uniform hemisphere sampling, we call `bvh->intersect` with this ray to get information about the intersection (filled in the `lightIntersection` object created right above), or to not add anything to `L_out` (in the event an intersection doesn't happen) because this ray isn't hitting anything in the scene and thus not receiving any light. We also perform the reflection calculation, which is virtually the same as in task 3 (although note the use of `abs_cos_theta` again, unlike in task 3's importance sampling!).

Our recursion stops in two ways. The first is if the ray depth is no longer above 0, in which case `max_ray_depth` (set in `raytrace_pixel`) has been reached, in which case we do not continue and return the `L_out` so far. The second is Russian Roulette. Russian Roulette provides an unbiased method of random termination. In effect, at every level of recursion, we can choose to terminate the recursion with some probability (we use 0.4 here). This is the purpose of the `coin_flip(CONTINUATION_PROBABILITY)` in the conditions for the `if-else` cases in the code; if either `max_ray_depth` has been reached *or* `coin_flip(PROBABILITY)`  is `false`, we terminate.

As explained in a previous section regarding Monte Carlo Estimation, dividing the result of the recursive call by `CONTINUATION_PROBABILITY` means we are effectively "boosting" the result by the equivalent amount to "nullify" the bias, and thus provide an unbiased estimate.

# Images Rendered with Global Illumination

![Image.png](Project%203-1%20Task%204.assets/Image.png)

![spheresGlobal.png](Project%203-1%20Task%204.assets/spheresGlobal.png)

# Comparison of Only Direct and Indirect Illumination

![Part4OnlyDirect.png](Project%203-1%20Task%204.assets/Part4OnlyDirect.png)

![Part4OnlyIndirect.png](Project%203-1%20Task%204.assets/Part4OnlyIndirect.png)

![Image.png](Project%203-1%20Task%204.assets/Image%20(2).png)

Top: only direct, Bottom: only indirect. **The black dots are a rendering artifact from the max depth of the ray being too low (3 in this picture). They do not exist if rendered with an input of 6 or greater to the argument `-m`. (picture correction!)**

Notice in particular the lack of any light on the ceiling due to the lack of bounces when only having indirect lighting, and the lack of any light from the area light when only using indirect lighting.

# Rendered Views with `max_ray_depth` at different levels

![Part4BunnyMaxDepth0.png](Project%203-1%20Task%204.assets/Part4BunnyMaxDepth0.png)

Rendered with purely Russian Roulette termination, theoretical infinity (i.e. `max_ray_depth` 0)

![Part4BunnyMaxDepth1.png](Project%203-1%20Task%204.assets/Part4BunnyMaxDepth1.png)

Rendered with `max_ray_depth` 1

![Part4BunnyMaxDepth2.png](Project%203-1%20Task%204.assets/Part4BunnyMaxDepth2.png)

Rendered with `max_ray_depth` 2

![Part4BunnyMaxDepth3.png](Project%203-1%20Task%204.assets/Part4BunnyMaxDepth3.png)

Rendered with `max_ray_depth` 3

![Image.png](Project%203-1%20Task%204.assets/Image%20(3).png)

Rendered with `max_ray_depth` 100

# Rendered Views at Different Sample-Per-Pixel Rates

![Part4SPPRate1.png](Project%203-1%20Task%204.assets/Part4SPPRate1.png)

Sample per pixel rate 1

![Part4SPPRate2.png](Project%203-1%20Task%204.assets/Part4SPPRate2.png)

Sample per pixel rate 2

![Part4SPPRate4.png](Project%203-1%20Task%204.assets/Part4SPPRate4.png)

Sample per pixel rate 4

![Part4SPPRate8.png](Project%203-1%20Task%204.assets/Part4SPPRate8.png)

Sample per pixel rate 8

![Part4SPPRate16.png](Project%203-1%20Task%204.assets/Part4SPPRate16.png)

Sample per pixel rate 16

![Part4SPPRate64.png](Project%203-1%20Task%204.assets/Part4SPPRate64.png)

Sample per pixel rate 64

![Part4SPPRate1024.png](Project%203-1%20Task%204.assets/Part4SPPRate1024.png)

Sample per pixel rate 1024. This was rendered with max ray depth 6. There are no artifacts like in the “direct vs. indirect illumination above”.

(Notice the noise reduction as samples-per-pixel increases! We’re allowing `colorForPixel` to converge when we take more samples per pixel).

