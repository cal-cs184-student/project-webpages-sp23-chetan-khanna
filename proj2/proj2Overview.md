# CS 184 Project 2 Overview

# Overview of Project

In this project, we implemented some basic Splining Techniques; in particular creating Bézier curves, by using the de Casteljau algorithm to map a set of control points into a smooth Bézier curve (and therefore a spline). We then extended this to surfaces (read the appropriate section for a conceptual elaboration on how this works).

We then implemented Area-Weighted Vertex Normals, which as I detail in the write-up, are used throughout shading (and indeed generally for anything “tilted”—through Lambert’s Cosine Law).

Finally, the bulk of this project consisted of implementing Loop Subdivision on meshes, including the prerequisites for it. This meant that we had to implement the ability to flip edges and split edges. Flipping edges was easier compared to splitting edges, which also involved adding elements to the mesh (my initial implementation did involve deletion of implements, which I later removed for performance purposes).

The Loop Subdivision algorithm itself was therefore the last thing I implemented; once the flip edge and split edge functionality was implemented, a lot of the more complex logic was removed out of it, and it was thus easier to create.

However, in my case personally, I discovered some difficult-to-catch bugs in the Loop Subdivision algorithm part of my code (which I detail in the appropriate section). While my flip edge and split edge were comparatively trouble free (although not entirely), this was not the case for my Loop Subdivision code. I detail this journey more, once again, in the appropriate section.

One thing this taught me was (yet again) the importance of reading code slowly while debugging. I had confirmed my logic was correct—indeed, I had actually got the logic for flip edge and split edge correct on the very first try because I drew a diagram and double-checked. However, I had accidentally made a typo, writing `h0` when I meant to write `h1`, which I missed even as I read through my code again.

That said, I found the final results I got from Loop Subdivision really interesting, and really enjoyed playing around with different ways of pre-processing the mesh to see how I could modify the “wireframe” at the beginning and predict what it would look like after several levels of Loop Subdivision.

