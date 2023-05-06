# Project 4 Task 5 (Including Extra Credit at End)

# Shaders

Shaders are utilities that let us perform calculations in bulk as we render our scenes on the display. Our shaders calculate two things: the positions of vertices of our object (i.e. positions of vertices on the mesh) or the color of pixels (not *exactly* pixels—but rather the outputs of the objects created by the shader's calculations of vertex positions).

Since this data has to be calculated in bulk (for every vertex on the mesh/for every “sample" we take), but is not necessarily big (shaders only return a vector), a GPU is ideally optimized to carry out these calculations quickly and in a highly parallelized manner. Shaders, therefore (in our case), use the GPU to perform these calculations. This enables us to speed up our render times significantly.

# Blinn-Phong Shading Model

The Blinn-Phong shading model is a model that allows us to compute the appearance of a surface based on the light in the scene. While it is not a *physically* correct shading model, it nevertheless approximates (and accounts for) a lot of the visual effects of light that we see.

The alternative approach, raytracing, is a newer technology and much more computationally expensive, so the Blinn-Phong shading model is actually one of the “default” ways of shading things (calcualting the appearance of a surface) in computer graphics today.

The model has three different components: ambient, diffuse, and specular. Each component is responsible for calculating a different visual effect, and these effects (components) are combined together to complete the model and thus compute a somewhat realistic appearance of a surface.

The ambient component is responsible for shading based on the “base” level of light (i.e. not on light sources). The diffuse component is based on light that comes from a light source but then is reflected out from the surface equally in all directions. The specular component is the most directionally-dependent, and responsible for the little bright spots (that move significantly based on the camera) from light sources you see on objects.

#### Ambient Only

![Ambient.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Ambient.png)

#### Diffuse Only

![Diffuse.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Diffuse.png)

#### Specular Only

![Specular.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Specular.png)

#### Entire Blinn-Phong Model

![Entire Blinn Phong.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Entire%20Blinn%20Phong.png)

# Texture Shader with Texture 3

![texture with texture 3.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/texture%20with%20texture%203.png)

# Bump Mapping and Displacement Mapping

### Bump Mapping vs Displacement Mapping

![bump.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/bump.png)

Bump Mapping.

![displacement.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/displacement.png)

Displacement mapping.

With bump mapping, we only modify the normal vectors according to the height map—while with displacement vectors, we take this a step further and also modify the positions of the vertices *themselves* (i.e. actually *change* the geometry of the mesh). Displacement mapping is less "fake" compared to bump mapping, therefore.

This is clearly visible in the results. The cloth isn’t “draping down smoothly” with displacement mapping, but rather “fluctuating” based on the texture we used. The exact correspondence of heights/final vertex locations can change based on the height map we are using; a different height map (for a given texture) will mean the cloth will rest (physically) in a slightly different way. I chose to use the `r` component of the color vector stored at the texture coordinates for the vertex we were operating on, as suggested in the spec.

### Coarse vs fine m

![bump 16.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/bump%2016.png)

![bump 128 2.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/bump%20128%202.png)

Bump mapping with coarse mesh (left), fine mesh (right)

![displacement 16.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/displacement%2016.png)

![Image.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Image.png)

Displacement mapping with coarse mesh (left), fine mesh (right)

Displacement mapping is able to take advantage of the finer mesh, since it changes the position of the vertices according to the height map (as explained above, bump mapping does not do this—it just changes the position of the normals and hence the *shading*, but not the *physical locations* of the vertices in space). This enables us to see a difference between the coarse and fine mesh. The difference is not nearly as pronounced with bump mapping.

# Mirror Shader

### On Cloth and Sphere

![mirror sphere cloth.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/mirror%20sphere%20cloth.png)

### On Sphere

![mirror sphere.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/mirror%20sphere.png)

### On Cloth

![mirror cloth.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/mirror%20cloth.png)

# Extra Credit: Custom Shaders

I combined the Blinn-Phong model with texturing in order to have a nicely lit effect, in addition to simulating some transparency based on the lighting itself.

The result of combining Blinn-Phong with texturing made the renderings of texture cloths look a little bit more realistic, reminding me of a slight “beach towel” look.

For implementing transparency, I did some research and it turned out to be easy: reducing the `alpha` parameter would reduce the opacity. At first, I reduced this to `0.5` uniformly. The first two screenshots show this.

Finally, I then made this “shading-dependent", so that the larger stronger the diffuse component in the Blinn-Phong shading model, the more "solid" the object would look. Because this was done in a "proportional" manner, this meant that the areas with stronger specular shading would actually appear to be more transparent.

#### Custom shader screenshots

![Image.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Image%20(2).png)

Notice the specular highlights from the light, indicating Blinn-Phong shading, on the textured cloth.

![Image.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Image%20(3).png)

Notice the cloth is slightly transparent at the bottom.

#### Custom shader with per-vertex lighting-based transparency

![Image.png](Project%204%20Task%205%20(Including%20Extra%20Credit%20at%20End).assets/Image%20(4).png)

Notice how parts that would typically have strong specular highlights are quite transparent; the other parts (which are mostly illuminated by diffuse lighting) are quite solid

