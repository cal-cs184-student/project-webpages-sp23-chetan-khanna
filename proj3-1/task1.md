# Project 3-1 Task 1

### Part 1

- Walk through the ray generation and primitive intersection parts of the rendering pipeline.
- Explain the triangle intersection algorithm you implemented in your own words.
- Show images with normal shading for a few small *.dae* files.

---

# Ray Generation and Primitive Intersection

### Ray Generation

The first task at hand was to generate a ray. This is the ray that will come out of the camera and be shot into the world (and then bounce off of objects—many of them), and we will collect the radiance of the ray (and modify based on BSDFs of the surfaces it hits). This will eventually become the irradiance of the pixel of the camera the ray was shot out from. (We take multiple samples, so actually the ray is shot out from this pixel multiple times and the value is averaged, but this is the overall idea of the concept).

For this to occur, `Camera::generate_ray` takes the normalized image coordinates $$(x,y)$$ as input. Since these coordinates were normalized, the first job is to find the equivalent coordinates on the sensor plane (in camera space).

More specifically, the position of the origin is the position of the camera (in camera space). The place on the sensor where a ray of light hits the camera’s sensor is where we see that ray’s impact on the image.

Therefore, given normalized image coordinates (i.e. a position on the *final* image), we needed to figure out where on the sensor (which is *not in normalized coordinates—it's in physical camera space*) that ray will be shot from.

(We are provided with `hFov` and `vFov` (field-of-view) of the sensor in camera space. It is clear that `hFov` and `vFov` are the field of view of the sensor. In this space, the camera is at the origin and the sensor is always at $$(0,0,-1)$$. As `hFov` and `vFov` become bigger, the angle subtended by the edges of sensor from the origin get bigger (hence your *field of view* getting bigger). These two values allow us to therefore figure out the relative *size* of the sensor, and are instrumental in converting from normalized image coordinates to sensor coordinates)

In order to do this, I calculated the bottom left point of the sensor in camera space (e.g. `double Norm0x = -tan(0.5 * hFovDeg)` and `double Norm0y = -tan(0.5 * vFovDeg)`). The top right of the sensor is given by the positive versions of these values. I choose the bottom left as $$(0,0)$$ because the normalized $$(x,y)$$ coordiantes are also relative to the bottom left of the image. Setting the same origin will let me just perform a "linear" scaling up based on the dimensions of the sensor (next paragraph).

I then calculate the length of each dimension of the sensor, which is just `tan(0.5 * <FoV for h or w>) * 2.0`.

Then, I can just scale up the provided normalized $$(x,y)$$ coordinates proportionally based on the sensor's size as explained above (e.g. `double xCoordinateInPlane = (double(x) * double(xLength)) + Norm0x;`) (it's linear—notice how it is similar to a $$\text{lerp}$$ from previous projects!)

I then create the actual vector from the camera to the point on the image plane. This is the direction vector of the ray, since we are shooting a ray from the *camera* (at the origin in camera space) through the *sensor* at the point $$(x_ \text{sensor}, y_\text{sensor})$$ (where $$(x_ \text{sensor}, y_\text{sensor})$$ corresponds to the point on the image $$(x,y)$$) into the world.

In order to move the ray into the world space, I simply multiply it by the rotation matrix `c2w`.

I then normalize the vector to length 1 since this is required for our ray (our `t` parameter for the ray, which is used to determine the position of the ray in time, would otherwise be different for almost every ray!).

I then create the `Ray` object with this vector. Since we shoot the ray from the camera, which is located at `pos` in world space, the origin of this vector is set to `pos`. Finally, the `max_t` and `min_t` are set to `fClip` and `nClip` respectively, which will be updated as the ray intersects with an object once it is in the scene (`min_t` is set to a non-zero value to avoid "self-intersections" with a primitive due to floating point error). This will be explained more in Task 3.

### Generating Pixel Samples

The conceptual explanations are mostly in the section above.

This function’s main job is to *actually shoot* the rays into the scene `num_sample` times for a given pixel, get the radiance that is "coming into” each ray (since we are shooting the rays *in reverse*, compared to what is happening physically) and then average the radiance from each of the samples to get the final value.

We shoot the rays into the scene “randomly” for this pixel; the only “constant” is that they “end up hitting this pixel” in the final image. We take multiple samples in random directions because physically, the light that is “making up” the irradiance of a single pixel is coming from all over the scene. We also need to take multiple samples because when the ray hits a surface, it can scatter off into one of many directions. Depending on the direction the ray takes, the path it travels (and therefore the later surfaces it hits, and therefore it’s radiance) can be significantly different. Taking multiple samples helps us reduce this “error” and each pixel in the image (and thus the image itself) “converge to it’s mode” and thus reduce noise.

We thus call `Camera::generate_ray(x,y)` with a random $$(x,y)$$ point. We get this point through `gridSampler->get_sample()`, process and normalize it (add to `origin` and divide by `width` or `height`) since `generate_ray`’s input takes normalized image coordinates. With this ray, we then call `est_radiance_global_illumination` to receive the radiance that this ray "contributes" to this pixel.

We add the radiance received from performing this process `num_sample` times (each time shooting the ray through a random different point). Finally, we divide the total radiance by `num_sample` in order to get the average radiance.

## Primitive Intersection

### Very Brief: Ray-Sphere Intersection

Ray-Sphere Intersection is performed by algebraically solving for $$t$$ by setting the ray equation equal to the sphere's (implicit) equation.

This leads to a quadratic equation. We check the discriminant first to test for intersection, and terminate early (and return `false`) if it is less than zero (this means there is no $$t$$ value of the ray at which the ray and sphere intersect, thus they never do).

We otherwise use the quadratic formula to solve for the intersection point, and choose the nearer of the returned points that still satisfy the criteria (above `min_t`, less than `max_t`, explained in more detail in the section below) as our intersection point. We choose the nearer since this is the first surface the ray hits; the ray will thus bounce off this surface and (effectively always) not the farther one. Further details (e.g. updating `isect`) are in the section below.

### Ray-Triangle Intersection

Triangle intersection occurs largely through the Moller-Trombone intersection algorithm. This algorithm is effectively just Cramer’s rule applied to solve a linear equation of the intersection of the ray and the triangle.

Since we want to find the intersection of the ray with the triangle, we set the ray equation equal to the triangle in barycentric coordinates. Rearranging algebraically into $$\text{Mx = b}$$ with unknowns $$\text{x}$$ being $$t, b_1$$ and $$b_2$$, we find a linear equation that can solve to give us the intersection point. In order to solve it, we apply Cramer's rule to find the solution to this equation. This general solution *is* the Moller-Trumbone algorithm; we just need to substitute in the numbers for our specific case to get $$t, b_1$$ and $$b_2$$.

In order to perform Moller-Trumbone, we first create the vectors of the two vectors for the two edges of the triangle (“originating” from the same vertex), the vector from the origin of the ray to that same vertex of the triangle, and the appropriate cross products. We then perform the rest of the calculations (see code) to get our solution vector $$\text{x}$$.

Note that we must check that this is *actually* a ray-triangle intersection after performing these calculations. First of all, we must check that the ray is actually hitting the triangle, which is easily done by checking that the barycentric coordinates of the triangle $$b_1, b_2, 1 - b_1 - b_2  ￼$$ are between 0 and 1 (and sum to 1), and that the ray is between `min_t` and `max_t` (`min_t` by definition is more than 0, being at least `nClip` upon generation).

If any of these conditions are false, then this ray doesn’t intersect this triangle. We immediately terminate and return `false`.

If this is the case, then we can set the `max_t` of the ray. The returned $$t$$ immediately gives us the value we can set the `max_t` of this ray to (which is necessary since we *know* that the ray *definitely* won't travel any further than this intersection; it's hit this object).

We also write to the `Intersection isect` object which is located at the address `&isect`. We know the normal of the intersection since we are given the normals at the vertices of the triangles and know the barycentric coordinates; barycentric interpolation simply gives us the normal, placed into `isect->n`. Naturally, we write `t` to `isect->t`. `isect->primitive = this` since we are in `Triangle` already, and we can call `Triangle::get_bsdf()` to get this triangle's BSDF function to place into `isect->bsdf`.

We finish by returning `true` since an intersection occurred.

# Images

![cow.png](Project%203-1%20Task%201.assets/cow.png)

Rendering of `cow.dae`

![Part1CBSpheresBox.png](Project%203-1%20Task%201.assets/Part1CBSpheresBox.png)

Rendering of `CBSpheres.dae`

