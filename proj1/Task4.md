# Task 4

# High-level intuition

Barycentric coordinates are a coordinate system based on specific “weights” connected to each of the three vertices of our triangle. We can use different sets of weights to uniquely describe a point in space, generally on the triangle. If a vertex’s has a higher “weight” associated with it, the point is going to be closer from that vertex.

# Definition

> More specifically, a point can be described by the following

> $$(x,y) = \alpha A + \beta B + \gamma C$$

> where  $$\alpha, \beta, \gamma$$ are weights corresponding to vertices $$A,B,C$$ of the triangle respectively.

Note that this means that, by definition, $$\alpha, \beta, \gamma$$ range from 0 to 1.

This structure means that we can easily define a property at the three vertices of a triangle, and then perform linear interpolation to get the “state” in all the other parts of the triangle (after all, it is just a matter of scaling the weights).

### Using Barycentric Coordinates for Interpolation, demonstrated with a triangle

The following image is an SVG courtesy of David Hägele, 2019 ([https://gist.github.com/companje/a54deb92be89269c10b4c5ca2b1a8a24](https://gist.github.com/companje/a54deb92be89269c10b4c5ca2b1a8a24)). It is placed on this webpage as a PNG for rendering, however:

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/781E0B2D-7CD5-42E1-8684-6E1B074183E1/9D35086B-E233-4C19-B254-DE764F685F3A_2/Xl9N5NyCrpsWQfyjmR8BqdYQ3izPFNqnDf906HiptH8z/Image.png)

The bottom-left vertex $$A$$ on this triangle is colored red, the bottom-right vertex $$B$$ is colored green, and the top vertex $$C$$ is colored blue.

Each point can be described by a unique $$(\alpha, \beta, \gamma)$$. Using this system we can therefore find our color at any point $$(\alpha, \beta, \gamma)$$ is given by $$ \text{color} = \alpha A_\text{color} + \beta B _\text{color} + \gamma C _\text{color}$$.

### Useful property

Barycentric coordinates have the following useful property:

> If $$\alpha + \beta + \gamma = 1$$ and all three weights are non-negative, then the point $$(x,y)$$ is inside the triangle.

This means we can convert a point during rasterization to barycentric coordinates (i.e. find $$\alpha, \beta, \gamma$$) and using the rule above quickly check if the point is contained within the triangle or not.

This means we no longer need to perform the half-plane intersection tests we did in previous tasks inside `rasterize_triangle`. Instead, after converting to barycentric coordinates, we just check if this property is fulfilled.

# In Code

In order to perform interpolation of color, I first converted each pixel into barycentric coordinates.

For a sample $$\text{p}$$, the conversion for $$\text{\alpha}$$ was performed by taking the cross product of vector $$\text{BC}$$ and vector $$\text{p - A}$$. The conversion was analogous for $$\text{\beta}$$. In order to minimize issues with floating-point error, $$\gamma$$ was defined as $$1 - \alpha - \beta$$. The usual half-plane line intersection test was nevertheless performed; it was done before computing barycentric coordinates and did not appear to reduce performance significantly.

After converting into barycentric coordinates, I calculated the color of the sample in the standard format $$\alpha A_\text{color} + \beta B_\text{color} + \gamma C_\text{color}$$, where $$A_\text{color}, B_\text{color}, C_\text{color}$$ are the colors of each of the triangle's vertices respectively. I put the result into the `sample_buffer` at the appropriate index (same as Task 2).

## Demonstration

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/781E0B2D-7CD5-42E1-8684-6E1B074183E1/BAF2E094-AC80-4DB6-A325-5CCEF9064CDB_2/xergtFD1iJdHrN3tp3U13gYA826OIhpvNi2JUi8y144z/Image.png)

