# Project 3-2 Task 1

## Screenshots

Place screenshots here

![T1RayDepth0.png](Project%203-2%20Task%201.assets/T1RayDepth0.png)

Maximum ray depth 0

![T1RayDepth1.png](Project%203-2%20Task%201.assets/T1RayDepth1.png)

Maximum ray depth 1

![T1RayDepth2.png](Project%203-2%20Task%201.assets/T1RayDepth2.png)

Maximum ray depth 2

![T1RayDepth3.png](Project%203-2%20Task%201.assets/T1RayDepth3.png)

Maximum ray depth 3

![T1RayDepth4.png](Project%203-2%20Task%201.assets/T1RayDepth4.png)

Maximum ray depth 4

![T1RayDepth5.png](Project%203-2%20Task%201.assets/T1RayDepth5.png)

Maximum ray depth 5

![T1RayDepth100.png](Project%203-2%20Task%201.assets/T1RayDepth100.png)

Maximum ray depth 100

## Changes

#### Basic Observations

- **Max Ray Depth 0:** we only see direct lighting
- **Max Ray Depth 1:** we now see indirect lighting but only one bounce (hence why we can see, for example, the reflections off of both balls, but not the refraction in the glass ball which requires an extra ray)
- **Max Ray Depth 2:** indirect lighting is now visible in the reflections of the balls (notice how we can perform a second "bounce" after reflecting off of the wall, and thus potentially sampling the light)
- **Max Ray Depth 3:** more indirect lighting (but not all) is visible in the reflections of the balls; the glass ball is much brighter (ray depth is high enough for refracted light to become visible)
- **Max Ray Depth 4:** the appearance of the glass ball in the mirror ball now appears “lifelike” and accurate, rather than black.
- **Max Ray Depth 5:** the glass ball appears *slightly* brighter.
- **Max Ray Depth 100:** the reflection of the area light off of the glass ball is simulated.

#### Explanation

At `max_ray_depth = 0` we only see direct lighting. This is because only the camera rays that hit a light will have a non-zero radiance. The reason for this is becuase we are not simulating light bouncing off of any surfaces yet (we're not shooting more rays and sampling the lights/integrating radiance at that point), so places like the balls and walls that have light only *reflecting* off of them simply register with a 0.

Increasing `max_ray_depth` from depth 0 to depth 1 introduces indirect lighting—in other words, light rays that emanate from the light can bounce off of the surfaces they bounce off of, and this radiance is summed (naturally "attenuated" as based on the BSDF function). More specifically, the radiance of the intersection points between the camera rays and objects in the scene is now higher because we now shoot rays from the intersection point and (in case of importance sampling) directly sample the point; since we didn't do this previously, these points just had a radiance of zero and thus looked black. (I elaborate below, when I took about the image rendered with `max_ray_depth = 3`)

Since `max_ray_depth` now allows us to cast rays from the sphere towards the light (and add to `L_out`, we are able to see the lights on the sphere. Note that the right sphere is also refractive, which is why the light appears less "smooth" and "shiny”.

Using the same logic, we see that `max_ray_depth` from 1 to 2 enables us to see some of the reflections of the indirect lighting on the ball; we can bounce again off the wall after having bounced off of the ball. This bounce enables us to accumulate the radiance from the area light at the top, which was not possible earlier.

From depth 2 to 3, we now see more indirect lighting effects on the mirror ball. In particular one can see the ceiling is now lit; this is because the ceiling receives no direct light from the area light, only light that is being reflected off of other surfaces, and we thus need to cast more rays for ceiling lighting effects to be visible on the ball (since reflecting off of the ball also “uses” some ray depth).

More significantly, the glass ball can also be seen more clearly. This is because we are now able to simulate the refractive effects of the glass ball, which we were not able to do earlier. In particular, we need to cast a ray from the beginning of the ball to the end (which “uses” one ray depth); this means, similar to “two-bounce rays” at, say `max_ray_depth = 1`, it was unable to reach a “place" where it could sample the lights and get a non-zero radiance.

At depth 3, there are enough bounces to reflect off of the mirror ball, onto the glass ball and onto the surfaces/walls near the ball. While it’s enough to hit the walls/other objects, it’s not enough to bounce off those walls to the light (which is the only source of light emission in the scene). This means that the glass ball appears mostly black, *except* when the ray reflecting off of the glass balls happens to hit the light directly (because that is when the radiance is non-zero). In more specific terms: the rays don't have enough remaining ray depth to be able to call `one_bounce_radiance` from a point *on the wall* (or any point a ray cast from the glass ball can hit, given it has already travelled from camera to mirror ball to glass ball). Thus, while we may end up "reaching" the walls, we can't actually recurse further (and thus call `one_bounce_radiance`) once we're there, and (because we multiply with the other terms once we return `L_out` at `max_ray_depth`), this “0" (or close to it) propagates up.

At depth 4, the ray depth is now high enough that *even* if a ray does not directly reflect off of the glass ball and onto a non-emissive surface, the ray depth is high enough that we are able to sample the lights. In other words, the ray depth is high enough to enable a ray to be shot from the camera, to the mirror ball, to the glass ball, to an object (e.g. a wall that is receiving light directly from the area light), and eventually hit the light. This means that the light that refracts through the glass is now 'visible’; since a large proportion of light hitting the glass ball was *refracted* (and not reflected), this makes the glass ball appear much brighter.

At depth 5, this effect is amplified (and thus the glass ball is slightly brighter) because those rays (that are being refracted) are able to bounce around more and thus are more likely to “reach” a point where `L_out` samples the light (or more of it), thus making the radiance higher. (This is analagous to what happened as we increased `L_out` at lower levels, only now the effect isn't directly visible so it is less obvious).

At depth 100, the ray depth is high enough that paths are very long. A ray from the camera that enters the glass ball can refract through it, go elsewhere in the world, hit it again and refract through it again, hit somewhere else in the world. Note that the amount of bounces that a ray undergoing total internal refraction also increases (thus the radiance being accumulated from this ray also increases).

