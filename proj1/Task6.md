# Task 6

When we are minifying textures—or drawing textures that are far away—one screen space sample corresponds to a large portion of texture space. This means that as we continue to take screen space samples and put them into texture space, we find that they are spaced very far apart (and therefore the texture samples we take are spaced very far apart). This, in turn, means that we are sampling texture space at a very low rate.

As noted in the section above, this can cause *aliasing*: our sampling frequency is much lower than the frequency of the signals of the texture space.

At the same time, we also want to *keep* higher-resolution textures, since if we want to draw a larger version of the image/the object we are drawing comes closer in our scene, then we would face a worse result (and have to perform interpolation in order to maintain an acceptable result).

We thus use *mipmaps*. Mipmaps store a texture at different resolutions, called *levels*.

For example, we store a (very) high-resolution texture at Level 0, e.g. 1024x1024. As we down one level, we halve the resolution in each dimension—going all the way down to 1 pixel at the lowest level! Mipmaps are relatively space-efficient: it turns out it doesn’t take a lot more space than just the original high-resolution texture image (one third more memory).

Depending on what our sampling of texture space is, we can choose an appropriate *level* of mipmap to reduce aliasing. (Intuitively, we want to choose the mipmap that has texture points aligning most closely with our sampling frequency).

We choose a level by looking measuring the distance between screen space samples for every screen space sample we take. I elaborate in [explanation of get_level](craftdocs://open?blockId=C906A990-6CF3-4AFB-BBB4-C1C1154D027C&spaceId=067f573b-b0da-d02e-e3ba-486aa57dc31a) below.

Note that we can also choose to use a technique to use *two levels* of the mipmap, since moving between each mip level could otherwise look jarring. In this case, for each screen sample we take, we actually take *two* (rather than one) texture space samples from two adjacent mipmap levels (derived from our normal mipmap level calculation). We then perform bilinear interpolation between the two to get our final texture space sample. This "combination" of two texture space samples roughly approximates continuous mipmap levels.

## Coding

For every screen space sample $$(x,y)$$ in `rasterize_triangle`, we find the barycentric coordinates in order to perform texture sampling (as usual). However, now, we also find the barycentric coordinates for $$(x+1, y)$$ and $$(x, y+1)$$ (in the same way). We construct (unscaled) vectors for each of these points, and we place them in a `SampleParams` struct.

We will use these extra barycentric coordinates soon, in T`exture,` in order to determine the distance between sample points in texture space, and thus decide on a mipmap level to use when sampling ($$(x,y)$$

We pass the `SampleParams` struct to `Texture::sample`. `Texture::sample` takes the pixel-sampling and level sampling settings (`psm` and `lsm`) into account. Unless `lsm` is set to `L_ZERO`, sample will call `get_level`, which returns the mipmap level (as a `float`) to perform sampling at.

#### `Texture::sample`: `lsm` set to `L_NEAREST`

If `lsm` is set to `L_SAMPLE`, then we round the mipmap level to the nearest integer and call the appropriate sampling method (`sample_bilinear` or `sample_nearest`).

#### `Texture::sample`: `lsm` set to `L_LINEAR`

If `lsm` is set to `L_LINEAR`, then we perform interpolation between the samples from two adjacent mipmap levels as described above.

In particular, to get a final sample, we:

1. Calculate `get_level` as normal. Note `get_level` returns a `float`.
2. We take the `ceil` and `floor` of the `level` in order to get two adjacent mipmap levels.
3. We take the sample from each of the two mipmap levels, returning a `Color`.
4. We perform linear interpolation based on the value of `get_level`. We use the fractional part of the `float` (i.e. remove the integer) to determine the weights between the two samples.
   - As an example: Say the original `level` was `2.9`. In this case, the fractional part is `0.9`, and thus the higher mipmap level will be weighted heavily, and the sample `Color` will be strongly weighted towards it.
1. We return the interpolated `Color`.

Note how, just like when performing linear interpolation in task 5, if `level` is an integer, the interpolation still occurs, except with a weight value of 0. This doesn't matter, since the "counterweight" is 1, and multiplies (by definition) with the same value.

Furthermore, note the stricter bounds checks performed here compared with the `L_LINEAR` case; this is necessary because `ceil(level)` must not exceed the highest mipmap level.

#### Explanation of `get_level`

`get_level` works by taking the $$\text{uv}$$ vectors corresponding to the $$(x+1, y)$$ and $$(x, y+1)$$ (and, as usual, scaling them up by `width` and `height`). Call these vectors $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$ respectively.

**Then:**

1. For each of $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$: Create a vector representing the distance between the original sample point in texture space (call this point $$(u,v)$$) and $$\text{uv_{x+1}}$$ (or $$\text{uv_{y+1}}$$ respectively).
   - Call the corresponding vectors $$\frac{d(u,v)}{dx}$$ and $$\frac{d(u,v)}{dy}$$. Note how $$\frac{d(u,v)}{dx}$$ is a vector showing how much $$(u,v)$$ change in texture space for a change of $$x$$ in screen space. (This is analogously true for $$\frac{d(u,v)}{dy}$$)
1. For each of $$\text{uv_{x+1}}$$ and $$\text{uv_{y+1}}$$: Use Pythagoras’ Theorem to find the length of each vector.
   - We split the vector $$\frac{d(u,v)}{dx}$$ into it's $$u$$ and $$v$$ components respectively, which are known. These form the legs of a triangle. The hypothesis of this triangle is given by Pythagoras' Theorem, and represents the length of $$\frac{d(u,v)}{dx}$$. (This is analogously true for $$\frac{d(u,v)}{dy}$$)
1. By convention, we take the maximum length between $$\frac{d(u,v)}{dx}$$ and $$\frac{d(u,v)}{dy}$$ to "represent" the distance between samples in texture space.
2. Our level of the mipmap will be the $$\text{log_2}$$ of this length; this is because each higher level of the mipmap is half the resolution, so doubling the length between screen space samples should lead to an increase of *one* mipmap level.

## Tradeoffs between speed, memory usage, and antialiasing power

| **Pixel sampling method** | **Level sampling method** | **Speed and memory usage (performance)** | **Antialiasing** |
| ------------------------- | ------------------------- | ---------------------------------------- | ---------------- |
| Nearest Pixel             | Zero                      | Best                                     | Worst            |
| Nearest Pixel             | Nearest                   | Good                                     | OK               |
| Nearest Pixel             | Linear                    | OK                                       | Good             |
| Bilinear Interpolation    | Zero                      | OK                                       | OK               |
| Bilinear Interpolation    | Nearest                   | Bad                                      | Good             |
| Bilinear Interpolation    | Linear                    | Worst                                    | Best             |

**Brief Comment:**

The fastest method is to use `L_ZERO` and `P_NEAREST`. This requires no calls to `get_level`, no interpolations performed anywhere, and no need to calculate sample points in texture space for adjacent points. However, it leads to significant aliasing, both of Moiré and Jaggies.

`L_NEAREST` and especially `L_LINEAR` will improve this but entail calls to `get_level`, the calculation of more vectors inside `rasterize_textured_triangle` and `get_level`. `L_LINEAR` will also lead to an interpolation operation inside `sample` itself, further increasing performance impact.

The best, of course, is `P_LINEAR` and `L_LINEAR` which will lead to reduced aliasing from sampling the texture (by enabling increased sampling frequency when sampling the texture), and also the choice of the correct mipmap (by enabling the texture to be of the best possible mipmap level, and therefore "frequency”)

## Demonstration with my own texture

The following picture shows `L_ZERO` and `P_LINEAR`:

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/46BF924A-B1B4-4BA6-9972-80936369611F_2/sgMHip2WAAOfM3g9dSyMtAMXLFCw0SaR5MvHrO9IuJsz/Image.png)

The following picture shows `L_NEAREST` and `P_LINEAR`:

![L_NEAREST P_LINEAR.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/3D8FDBBD-4622-4D87-AF57-C6CD022B2411_2/JUhEOLUHdNugb7BBZAmXPatkRD2lHEH3T7Ey20GZesIz/L_NEAREST%20P_LINEAR.png)

The following picture shows `L_NEAREST` and `P_NEAREST`:

![L_NEAREST P_NEAREST.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/C52E0B7A-B1E2-4E48-B52C-E1209CF3FEC2_2/Lvj1cr23O7k5gS9EDzzNRk1SBajqhcQiZ1zPT6pWyU4z/L_NEAREST%20P_NEAREST.png)

The following `L_ZERO` and `P_NEAREST`:

![L_ZERO P_NEAREST.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/EBBD1BE1-FABE-4FDB-8913-D37EB84A0C85_2/pBp92TG93QdT12ITrcwsbRGxr3nxFBc3h3D3KC9L0tkz/L_ZERO%20P_NEAREST.png)

## Demonstration with class-provided picture

In order to demonstrate the effects more clearly, the following shows `test4.svg`:

`L_ZERO` and `P_NEAREST`:

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/09218719-59E6-4814-A126-FF38CA0A1E6B_2/0PLqVBqDhVVZA3IsYKURyoy0Socu60MRb4ceNobIAAwz/Image.png)

`L_ZERO` and `P_LINEAR`:

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/3587E92C-3569-4032-AC16-99A00CC7759C_2/uJH3vUOn9049SwnObx1qMfRmPkP0825XqlskbXd7X74z/Image.png)

`L_NEAREST` and `P_NEAREST`:

![Image.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/A0744F1C-7997-46C2-91DA-5CAFB744C151_2/eExq2jDvxvytqEzYnlI2XMYWP9LgMEhyQkyqzyMiaNkz/Image.png)

`L_NEAREST` and `P_LINEAR`:

![l nearest p linear logo.png](https://res.craft.do/user/full/067f573b-b0da-d02e-e3ba-486aa57dc31a/doc/6DC693ED-D5D1-4026-AAB3-50B44315177A/255E109E-145F-4852-8CAD-052F9D7DEF8B_2/Ao4g3dxzidH1vy5sxxtPFwVJu6Lk1v7hi2CRWxvaGLgz/l%20nearest%20p%20linear%20logo.png)

