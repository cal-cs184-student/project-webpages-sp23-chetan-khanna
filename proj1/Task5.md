# Task 5

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

### A note on bilinear vs nearest-pixel sampling

Because we are *sampling* texture space, all the usual caveats of sampling apply. Most importantly, this includes aliasing. Aliasing is most clear in the cases of texture magnification or, especially, texture minification.

Bilinear interpolation, as described above, is in effect an antialiasing technique. While the nearest pixel technique is only taking a single sample of the texture for each screen space sample, the bilinear interpolation strategy takes the *four* closest texture points and interpolates between them to return a single value. This means we sample at a higher frequency at a higher frequency and thus acts as an antialiasing technique.

### Demonstration of differences

The following shows nearest pixel sampling at `sample_rate` 1.

![nearest 1.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/198A565C-0D98-4906-9AEC-D53ABCA4A521/642B69D3-40A6-4640-97E9-9A446BF38993_2/kf9STvBawouXyYZjFkrdBNb91RcorxNMqXNcrzQEqMYz/nearest%201.png)

The following shows pixel sampling using the bilinear interpolation scheme at `sample_rate` 1.

![bilinear 1.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/198A565C-0D98-4906-9AEC-D53ABCA4A521/FBCACB75-60F4-4FDB-A1FF-26AF87892EA9_2/ETqcbl90aMyFcGqb61uB2nqfz6UT4qRWMBs0d9x1g84z/bilinear%201.png)

The following shows nearest pixel sampling at `sample_rate` 16.

![nearest 16.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/198A565C-0D98-4906-9AEC-D53ABCA4A521/AB24119D-92F0-4228-A04D-96F9CF72EA25_2/8Wd7ykfP88bloKXIsVIlo95pnLAgYWALsppDjPyX9uIz/nearest%2016.png)

The following shows pixel sampling using the bilinear interpolation scheme at `sample_rate` 16.

![bilinear 16.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/198A565C-0D98-4906-9AEC-D53ABCA4A521/3E1B77A5-A3CB-4A02-AD97-887AC7DFFC67_2/9WMbKqsVMvfgvytPXZmtMOQQML4bwPgxKdyx2lMbYQ4z/bilinear%2016.png)

### When there is a large difference

I discussed this above already. To summarize: the bilinear interpolation scheme is most helpful when aliasing from texture sampling is maximized. This effect is *most* clear when the texture is highly minified, because our screen space samples occur at a much lower frequency in texture space. This means we are sampling our texture space a lot less frequently, meaning aliasing is most likely to occur.

Naturally, this effect is easier to encounter when the texture itself is composed of faster moving signals, such as many distinct shapes, or scenes like tree branches. In order to encounter *no aliasing at all,* we need to sample at at least twice the frequency of the fastest-signal in the texture (this relates to the *Nyquist frequency*), so a texture with faster signals means this effect is more likely to occur for a given sampling frequency.

