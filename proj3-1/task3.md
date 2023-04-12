# Project 3-1 Task 3

# Implementations of Direct Lighting Function

In this section, we are trying to estimate how much light arrives at a point in the scene. Specifically, we care about the point `hit_p` where the ray we shot from the camera is intersecting with the surface it hit. This is necessary because we need to determine how much light (and the spectrum of it, since we’re dealing with colors) is reflected back towards the camera from this point (this is, after all, the point of shooting the ray, albeit inversely, through the camera). Thus in both functions our aim is to use the reflection equation to determine how much light is being reflected from `hit_p` back through the ray towards the camera.

## Uniform Hemisphere Sampling

With Uniform Hemisphere Sampling, we calculate how much light is reaching a point `hit_p` by integrating over all the light arriving in a hemisphere around this specific point (in other words, basically adding up all the light hitting this point).

Because this is very computationally expensive, we can use Monte Carlo Sampling in order to estimate the integral without actually having to integrate.

To find all the light reaching this point, we need to shoot another ray from this point back into the scene (in a uniformly random direction over the hemisphere, i.e. with $$\text{pdf} = 1/ 2 \pi$$), and find how much light is coming from this point. Since we are only dealing with direct lighting in this part, this *is not recursive,* but rather just checking the direct light emissions coming from this "shot-out" ray (i.e. it will only be non-zero if the ray hits a light source).

Because we are Monte Carlo sampling, we must take shoot the ray out `num_samples` times in a uniformly random direction, using the reflection equation to find the light being reflected back from this point to the original ray each time. We also divide by the $$\text{pdf}$$ in order to avoid bias in the Monte Carlo estimator (the logic is that rays that are shot out in a direction with a higher probability of being selected contribute less to the value and vice versa).

At the end, we average the total light by the number of samples (as per the Monte Carlo estimator)—we’d end up with a very bright (and incorrect) value otherwise!

#### Specifics

- In order to shoot a ray from the intersection point outwards randomly, we use `hemisphereSampler->get_sample()`, which gives us an outgoing (i.e., what is actually "incoming to the point") ray that is uniformly random in "intersection” coordinates (i.e. origin is intersection of the point with normal of intersection point being the $$z$$ basis vector).
- The ray we shoot back is in world coordinates, so we convert the sample direction vector we got from the sampler to world coordinates (multiply by `o2w`). Furthermore, we set `min_t` to `EPS_F` otherwise due to floating-point error the ray may intersect with the surface it had just intersected with (e.g. if floating-point error causes the point to be considered slightly "inside" the surface).
- We call `bvh->intersect` for each ray we shoot out from the intersection point. This is for two reasons.
   1. If `bvh->intersect`, then we know this ray doesn't intersect anything in the scene (i.e. a light) and thus we don't add to our lighting estimate.
   2. `bvh->intersect`, if it finds an intersection, writes details of the intersection to the passed in `Intersection` object. This passed in `Intersection` object will contain the details about the object it hit. We will use these details to find out the emission of this object, which is obviously necessary to calculate how much light is hitting our intersection point of the camera ray!
- When calculating the reflection equation, we get the `Vector3D radianceOfLightSource = lightIntersection.bsdf->get_emission()` (which will be 0 unless it is emitting light; i.e. the ray is hitting a light), `Vector3D reflectivityOfCurrentSurface = isect.bsdf->f(w_out,w_j_object)` (notice how we call `isect`, not `lightIntersection`,  since we're looking at the original intersection point with the camera ray of the current surface), and `double cosineTerm = cos_theta(w_j_object)` (since a steeper angle between the ray the light is coming from and the normal, the lower the actual light; think Lambert's Law). We multiply these terms together (and, as mentioned above, divide by the `pdf`)

## Importance Sampling of Lights

One of the big conceptual differences between importance sampling and uniform hemisphere sampling is that we directly sample the light sources. This means we shoot out rays in the directions that are hitting lights (and thus sampling them) directly. This is in contrast to uniform hemisphere sampling, where we just shot them out in uniformly random directions in the hemisphere.

This is a lot more efficient (thus enabling less noisy pictures); at the end of the day, the only contributions made to direct lighting occur by objects that are emitting light (i.e. lights!). Directly sampling them means we’re wasting computing resources on rays that will never contribute to the light at the intersection point `hit_p`.

While the code of this function looks similar to above, it’s conceptually very different.

Firstly, for each light, we perform the full Monte Carlo estimate, sampling it `num_samples` times (unless it is a point light) and then normalizing it by `num_samples` (notice we do not normalize after having sampled *all* lights). In other words, we sample light-by-light, estimating how much each light is contributing to the light at `hit_p`.

Furthermore, to get the radiance of a light source, we (of course) shoot out the ray directly to a point on the light we are currenty sampling using `thisLight->sample_L`. Among other things, this gives us the direction from the point to the light source (`w_j_world` in the code).

However, this does not check if something is blocking the path of light from this point on the light source to `hit_p`! We therefore cast a ray from the point to the light. We check for intersection using `bvh->intersect`; if this returns `true`, it indicates that the ray of light has hit something in the scene, and don't add the value returned by `sample_L` to `L_out`. (Notice how the purpose of this ray is *very different* to uniform sampling!)

We use `cos_theta` (but not `abs_cos_theta`) to calculate the cosine term in the reflection equation. However, notice that `sample_L` can sample a point on a light *behind `hit_p`, i*n which case the contribution should be 0 since the light cannot travel through the object `hit_p` is on. (A scenario obviously not encountered when you are sampling a hemisphere *above* the point). Thus if `cos_theta(w_j_object)` is less than 0, we also don't add to the light's contribution.

We get the `Vector3D reflectivityOfCurrentSurface = isect.bsdf->f(w_out,w_j_object)` in identical fashion to uniform hemisphere sampling.

Note that the $$\text{pdf}$$ is returned by `sample_L`, which means it is easy to divide by it for each sample we take (as explained above, as part of the Monte Carlo estimate).

(Side note: I don’t mention the conversions from world to object coordinates and vice versa in this section; they should be intuitive by now, as can also be noted in my code comments.)

# Images Rendered With Both Implementations of the Direct Lighting Function/Comparison Between Both

*(The explanations at the beginning of this section make clear why importance sampling leads to less noisy results. I don't elaborate on it conceptually.)*

As is expected, importance sampling leads to significantly less noise and much softer shadows.

#### Bunny: Uniform Sampling

![Image.png](Project%203-1%20Task%203.assets/Image.png)

#### Bunny: Importance Sampling

![Image.png](Project%203-1%20Task%203.assets/Image%20(2).png)

#### Spheres: Uniform Sampling

![Image.png](Project%203-1%20Task%203.assets/Image%20(3).png)

#### Spheres: Importance Sampling

![Image.png](Project%203-1%20Task%203.assets/Image%20(4).png)

# Images With At Least One Area Light and Noise Level Comparison

![Image.png](Project%203-1%20Task%203.assets/Image%20(5).png)

![Image.png](Project%203-1%20Task%203.assets/Image%20(6).png)

![Image.png](Project%203-1%20Task%203.assets/Image%20(7).png)

![Image.png](Project%203-1%20Task%203.assets/Image%20(8).png)

Top: 1, second from top: 4, second from bottom: 16, bottom: 64

Taking just one sample per area light leads to poor results because `sample_L` only gets to sample very few points on the light. It's possible that sometimes some of these rays are blocked by an object in the scene. With such few samples, the impact of these few samples is significant on the final light value of the pixel. In other words, each pixel has not had enough samples to be able to converge to a more representative value of the actual light falling on it—if `sample_L` happened to sample a point on the light which was blocked by another object in the scene (thus causing a shadow), we're not taking another (or enough) samples to "normalize" the effect of this.

