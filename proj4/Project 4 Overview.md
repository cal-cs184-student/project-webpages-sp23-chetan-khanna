# Project 4 Overview

In Project 4, we simulated the physics of a cloth based on a “mass-spring” philosophy. The cloth was represented by a mesh of vertices connected together by springs (of three types, each serving a different purpose: structural, shear, and bending). We simulated forces on these springs (and thus this grid) using physical laws (such as Hooke’s Law) in order to try and realistically simulate the shapes and position of the entire cloth and visualize it.

For this, we used Verlet integration. In order to try and keep the simulation stable, we used Provot’s method to constrain the position of the springs at any time step. This prevented the springs from “stretching out” just a little too far at some time step (which would eventually “balloon” as that addition force gets passed to the next spring) and the entire simulation “blowing up“.

We then implemented the ability for the cloth to intersect with spheres and planes in the scene. This enabled us to see how the cloth behaved when it collided with other objects.

After this, we also implemented the ability for the cloth to collide with itself. Since there was no checking for self collisions thus far, parts of the cloth could intersect and go through each other, which was physically unrealistic. In order to do this efficiently, we divided the cloth spatially into different “buckets” (based on a hash function we created) and put each vertex into a “bucket” (entry of our hashmap), and then for every point checked for collisions against the other points in that “bucket”. We then corrected points that were colliding. This improved the physical realism of the cloth—for example, enabling the cloth to fold on top of itself realistically when it collided with a plane.

The last part of the project was implementing shaders that allowed us to color the cloth in different ways. We implemented Blinn-Phong shading and Texture Mapping. We also implemented Bump and then Displacement Mapping, which used height maps to modify the normals and thus make the surface *look* bumpy. Displacement mapping went a step further: it made the surface *actually be* bumpy based on the height map by modifying the positions of the vertices. Our final use of the shaders was for environment-mapped reflections, allowing the cloth and sphere to effectively "mirror" the environment around us.

