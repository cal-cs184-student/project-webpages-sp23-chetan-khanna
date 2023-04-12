# Project 3-2 Overview

In this project, we extended our raytracer to simulate more complex materials and their interaction with light. This was primarily done by modifying their BSDF functions.

I implemented Parts One and Two in this project.

In part one, we implemented mirror materials, which involved implementing perfect specular reflection (which is not *only* used for mirrors; it is actually in the `BSDF` base class). The more interesting and rewarding part of part two, however, was implementing refraction (for glass materials).

For this, we needed to implement refraction, since light that goes through glass gets refracted. We used Snell’s Law for this. Furthermore, since some light is refracted while some is reflected, we also implemented a function that can *approximate* the ratio of light reflected to light refracted. Based on this, we would choose to either *reflect* the current ray of light or *refract* it.

Finally, Part Two was primarily about implementing microfacet materials. This approach looked mostly at the “roughness” of materials as based on the distribution of their facets (height of the surface at the micro-level). Since light is transmitted and reflected very differently based on the distribution of facets (the directions of their normals, particularly), this has a big impact on what a material looks like (and correspondingly on the scene). In this section, we therefore implemented several aspects of microfacet materials, including the BSDF evaluation function and its constituent parts. Note that here we are actually calculating the Fresnel term, compared to Part One where we simply used Schlick’s approximation. Schlick’s approximation is not usable for the microfacet materials we are modeling. We then implemented importance sampling for these materials, which enables us to cast rays in outgoing directions based more correctly on the actual distribution of the facets of the microfacet material (i.e. on it’s actual texture and material).

---

**Website access:** [https://cal-cs184-student.github.io/project-webpages-sp23-chetan-khanna/proj3-2/project32Access.html](https://cal-cs184-student.github.io/project-webpages-sp23-chetan-khanna/proj2/project2Access.html)

