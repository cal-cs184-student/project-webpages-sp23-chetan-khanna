# CS 184 Project 2 Task 1

# Brief Explanation of de Casteljau’s Algorithm

de Casteljau’s algorithm, in 1D is a method to create a Bézier curve from a series of “*control points*” that we specify.

In a nutshell, the algorithm works by recursively performing linear interpolation between all the control points it encounters in each recursive level. It will eventually reach a state where there is only one line segment (one set of two control points) to interpolate. The point returned by this final interpolation is a point on the Bézier curve.

We do this with different $$t$$ values to get different points on the Bézier curve, and thus draw the curve.

Note the recursive nature of this algorithm: every level reduces the number of points to interpolate between by $$1$$—and thus the line segments that connect each point to the next point by $$1$$ also.

### My Implementation

In order to perform a single step (level) of recursion—i.e. drop $$n$$ control points to $$n-1$$ points, I take a `std::vector` of control points (formatted as `Vector2D` objects) I've received, and iterate through them, taking the $$i^ \text{th}$$ and $$(i+1)^ \text{th}$$ control point and performing a standard linear interpolation between them:

$$\text{interpolated point = (1-t) \times point_i +  t \times  point_{i+1}}$$

where $$t$$ is the interpolation parameter (provided), and $$\text{point_x}$$ is a `Vector2D` object that stores a vector that refers to the position of $$\text{point_x}$$

# Bézier Curve with 6 control points

These screenshots show a Bézier curve with six control points I chose. As you go down you will see the same Bézier curve but with another step of interpolation done.

![Screenshot 2023-02-22 at 17.00.42.png](CS%20184%20Project%202%20Task%201.assets/Screenshot%202023-02-22%20at%2017.00.42.png)

![Screenshot 2023-02-22 at 17.01.24.png](CS%20184%20Project%202%20Task%201.assets/Screenshot%202023-02-22%20at%2017.01.24.png)

![Screenshot 2023-02-22 at 17.01.29.png](CS%20184%20Project%202%20Task%201.assets/Screenshot%202023-02-22%20at%2017.01.29.png)

![Screenshot 2023-02-22 at 17.01.38.png](CS%20184%20Project%202%20Task%201.assets/Screenshot%202023-02-22%20at%2017.01.38.png)

![Screenshot 2023-02-22 at 17.01.45.png](CS%20184%20Project%202%20Task%201.assets/Screenshot%202023-02-22%20at%2017.01.45.png)

![Image.png](CS%20184%20Project%202%20Task%201.assets/Image.png)

# Bézier Curve with moved control points and changed $$t$$

![screenshot of slightly different Bezier curve.png](CS%20184%20Project%202%20Task%201.assets/screenshot%20of%20slightly%20different%20Bezier%20curve.png)

