# CS 184 Project 2 Task 2

# Extension of de Casteljau to Bezier Surfaces

The extension of the de Casteljau surfaces is fairly logical, and indeed reminds me somewhat of pixel sampling using bilinear interpolation. With pixel sampling using bilinear interpolation (in Project 1), we had four points (each containing a `Color`) that we wanted to reduce to a *single* point to return. For this we performed *two* linear interpolations to "collapse” the 2D system of 4 points (all at different $$(x,y)$$ into a 1D system, containing two points (at the same $$x$$’s, but the same $$y$$, or vice versa).

Extending de Casteljau to surfaces uses somewhat similar logic. We perform the de Casteljau algorithm across all of the $$n$$ Bézier curves that "travel" unchangingly across 1 “axis” of the surface first (which we can call the $$u$$-axis) (their values may change, but they stick to the same $$u$$ value across the entire surface). Once we have done this, we are left with $$n$$ points in space, each the result of performing de Casteljau on those Bézier curves. We can consider these $$n$$ points now to *control* points on *another* Bézier curve, travelling across the surface—not necessarily in that 1 dimension.

We *then* perform de Casteljau—just like we do normally—but using these $$n$$ points we've found as control points for a Bézier curve. This collapses the $$n$$ points down to $$1$$ point, which is the point on the Bézier surface.

## My Implementation

#### Functions used

I use several functions in my implementation:

- `evaluate`: main "driver" function. Returns the final point on the Bézier surface.
- `evaluate1D`: helper for `evaluate`. Returns the final evaluated point on a Bézier curve.
- `evaluateStep`: helper for `evaluate1D`. Performs one step of interpolation, reducing $$n$$ points to $$n-1$$ points.

#### Flow

`evaluate` is the main driver function. It first creates a `std::vector<Vector3D> firstInterpolations` for storing all the points that will come from performing the interpolations across the first $$n$$ fixed-$$u$$-axis traveling curves. It then calls `evaluate1D` (elaborated upon below) on each curve individually (always with interpolation parameter `u`). This is done by indexing into each row of the input 2D vector of 3D points, `BezierPatch::controlPoints`and calling `evaluate1D` iteratively, and adding the returned `Vector3D`, representing the final interpolated point for that Bézier curve, to `firstInterpolations`. After this, it calls `evaluate1D` once again—but, naturally, on the vector `firstInterpolations` and with interpolation parameter `v`. This returns a final point which is the evaluated point on the Bézier surface.

`evaluate1D`’s job is to perform the de Casteljau algorithm on an input curve and return the final interpolated point. `evaluate1D` calls `evaluateStep`, which performs one step of the de Casteljau algorithm (like in part 1, except with `Vector3D` rather than `Vector2D` objects and the passed in `t` parameter) until the `vector` of points returned by `evaluateStep` is of length 1. Once the vector returned by `evaluateStep` is of length 1, this would indicate that the de Casteljau algorithm has performed the final interpolation between two points and is only returning a single point. `evaluate1D` returns the point contained inside this `vector`.

`evaluate1D` performs this action repeatedly by first creating a `std::vector interpolatedPoints`, which is set to the passed in `points` upon initialization. Then, in a `while` loop, it calls `evaluateStep` and sets the result of `evaluateStep` to the `interpolatedPoints` vector in each iteration. The `while` loop terminates once `interpolatedPoints` is a length 1 vector.

# Demonstration: Teapot Screenshot

![Screenshot 2023-02-22 at 18.40.08.png](CS%20184%20Project%202%20Task%202.assets/Screenshot%202023-02-22%20at%2018.40.08.png)

![Screenshot 2023-02-22 at 18.40.21.png](CS%20184%20Project%202%20Task%202.assets/Screenshot%202023-02-22%20at%2018.40.21.png)

