# Project 3-2 Task 2

# Impact of $$\alpha$$ values on rendered images

Decreasing $$\alpha$$ values from top to bottom (highest at top, lowest at bottom)

![T2Alpha0.5.png](Project%203-2%20Task%202.assets/T2Alpha0.5.png)

![T2Alpha0.25.png](Project%203-2%20Task%202.assets/T2Alpha0.25.png)

![T2Alpha0.05.png](Project%203-2%20Task%202.assets/T2Alpha0.05.png)

![T2Alpha0.005.png](Project%203-2%20Task%202.assets/T2Alpha0.005.png)

As we decrease the $$\alpha$$ value, the reflections of light on the dragon tend to become more "localized" and clearly visible, and the dragon tends to get somewhat darker overall. Furthermore, we see very bright spots appear across the image (artifacts).

These differences make sense; decreasing the $$\alpha$$ value increases the smoothness of the surface. This means that as the $$\alpha$$ value is decreased, the distribution of the microfacets' normals is more concentrated. Therefore, when light from the area light hits the dragon, it no longer "scatters" (and brightens the *entire* dragon) like it did before; it simply tends to reflect off of the area of the dragon it hit itself.

The more extreme and localized shadows are probably what cause the “sparkly” artifacts; if, during the pathtracing of a ray, one of the “deeper” rays hits one of these localized bright spots, we will “carry up” this high radiance, at least partially, back to the original ray.

# Cosine Hemisphere vs Importance Sampling Images

![T2CosineSamplingBunnyCU.png](Project%203-2%20Task%202.assets/T2CosineSamplingBunnyCU.png)

Cosine hemisphere sampling of bunny

![T2ImportanceBunnyCU.png](Project%203-2%20Task%202.assets/T2ImportanceBunnyCU.png)

Importance sampling of bunny

Cosine hemisphere sampling is noisier than importance sampling. This is particularly true for the bunny itself, where the difference is “obvious”. This is not hugely surprising; the directions are cast according to the cosine-weighted hemisphere sampler’s $$\text{pdf}$$ rather than directly towards the light. Furthermore, since they're not taking into account the actual microfacets on the surface, the cast rays are not representative of how the light would *actually* reflect off of the surface (due to its microfacets).

With importance sampling, on the other hand, we are explicitly are choosing $$\theta$$ and $$\phi$$ to resemble $$D(h)$$ quite closely—in other words, we are *choosing* the outbound ray direction *based* on the microfacets of the material. This means we actually represent the material properly with our shading. In other words, our implementation using importance sampling is taking the actual distribution of the normals of the microfacets of the material into account, hence the more natural (and noise-free) representation.

## Personal Material

This is a rendering with the (admittedly rather bizarre) choice of a human liver.

- `eta`: 1.3770 1.3822 1.3850
- `k`: 0.0034630 0.0036333 0.0037307

![Image.png](Project%203-2%20Task%202.assets/Image.png)

