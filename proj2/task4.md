# CS 184 Project 2 Task 4

# Brief Explanation: Implementation of Edge Flip Operation

The edge flip operation really boils down to two things:

1. Understanding the geometry of the edge flip
2. Reassigning pointers correctly based on that understanding of geometry

#### Explanation

Understanding the geometry of the edge flip is easily done with a diagram.

In addition to the obvious benefit a diagram brings (you can trace what happens to each and every element in the half-edge data structure), it also brings one other important realization: *every edge flip has the same geometry, so an edge flip is a very "generalizable" operation.* In other words, we don't have to care about many edge cases (and assign pointers differently for each edge case)—the elements flip in the same way.

To trace what happens to each element after an edge flip, I used the resources available on the CS 184 website, and in particular, the “Guide to Implementing Edge Operations on a Halfedge Data Structure” (CMU CS 15-462, Fall 2015, [http://15462.courses.cs.cmu.edu/fall2015content/misc/HalfedgeEdgeOpImplementationGuide.pdf](http://15462.courses.cs.cmu.edu/fall2015content/misc/HalfedgeEdgeOpImplementationGuide.pdf)) linked on the CS 184 course website.

It provided the following diagram, which made the edge flip operation distinctly clear.

![Image.png](CS%20184%20Project%202%20Task%204.assets/Image.png)

Clearly, we are flipping edge `e0`, so it is connecting `v3` and `v2` correctly instead of `v0` and `v1`. ***Note the point I said above: while not every mesh structure will be "physically" the same (e.g. same dimensions, equilateral triangles, and so on—every mesh structure will be "relatively" the same—e.g. the `twin` of a halfedge will always be on the other side of the mesh, in the other triangle, or that calling `thisHalfedge->next()->next()->next()` will lead to `thisHalfedge` again, since we are always dealing with triangles.** This makes it clear that this diagram is generalizable for every (non-boundary) edge flip.*

The same source also provides a general *structure* for how to attack the problem (which was a little bit more obvious).

We first setup pointers to every element in the half-edge data structure. This is easily done because we are given `e0`. We can access `e0`’s half-edge directly, and since half-edges link the entire mesh together, we can access every other element. Therefore, having gotten `HalfedgeIter h0 = e0->halfedge()`, we can call `next` on `h0` to get `h1` (and so on), as defined in the diagram. (The actual variable names follow the diagram above; they are otherwise unimportant.)

We then reassign pointers for all the elements in the mesh. *Because we had previously assigned pointers to elements and stored them in variables such as `h5`, `e1`, `v3`, `f1` and so on, and because we are changing the assignments "all in one go" (i.e. one operation is "atomic*” *in nature, we can't do half a flip then a split then complete the remaining half of a flip)*, there is no issue with reassigning new elements to the "old version” of a previous element (these are all *pointers*, after all).

The primary pointer changes happen to the half-edges, which makes sense since half-edges are the elements linking the mesh together (i.e. enabling traversal of the mesh) and thus have many pointers (e.g. to the `twin` and `next` half-edges). I reassigned all half-edges’ `next`, `twin`, `vertex`, `edge`, and `face` pointers starting from `h0`. Notice how `h3`'s (source) `vertex` is now `v2` and `h0`'s `vertex` is `v3` now, indicating the flipped edge (we will later reassign `e0` to have one of these half-edges).

*As a side note:* note how we must change some pointers in `h6`, `h7`, `h8`, and `h9`. Most of the pointers for these half-edges remain the same, but their `twin` pointers change. Note also how because they are on the "outside" we never actually set pointers to their `next` or `face`. Since their `next` and `face` pointers are always unaffected, we just call `hx->next()` and `hx->face()` to get them as we are calling their `setNeighbors` method.

Now that we have done the “hard work", setting pointers for most of the other elements is pretty simple—they just link to a half-edge (once again, the `halfedge` is what does the real work of linking the mesh in the half-edge data structure). Thus, for all vertices, edges, and faces, we just assign them to their new half-edges, thus placing these elements in the right "location" in the mesh. In a sense, the "real work" was done by moving half-edges around, and now we're just putting these other elements in the right place by linking them to the right half-edges.

As a side note, note how we had some choices for elements: all the edges, for example, could point to any half-edge they were associated with. There were also some elements that *potentially* could have been left alone *if* they happened to point to an "unaffected" element—for example, if `v1` pointed to `h9`, then we would not need to reassign `v1->halfedge()`. However, since it is likely that at least some of these elements *do* point to an affected element (e.g. `v1` could equally well instead point to `h3`, which would be valid before the flip but invalid after), we reassign them to a valid `halfedge` anyways.

# Screenshots of Teapot Before and After

#### Before edge flips

![Image.png](CS%20184%20Project%202%20Task%204.assets/Image%20(2).png)

#### After edge flips

![Image.png](CS%20184%20Project%202%20Task%204.assets/Image%20(3).png)

# My Eventful Debugging Journey

I went over my code carefully as I was writing this since I didn’t want to re-assign pointers incorrectly—that would be hard to debug!

The “only” issue I experience was making a typo when initially setting up my pointers to the mesh before I did the flip. In particular, I assigned `h2 = h0->next();` which was obviously incorrect (and identical to my assignment for `h1`). This meant `h1` and `h2` pointed to the same thing! `h2` should have been `h1->next()`. I missed this typo when reviewing my code.

The only clue I had before this was visual; there was a semi-demented “edge collapse" occurring, which indicated to me that a pointer somewhere was being lost. Calling `check_for` on the reassigned pointers was not all that useful, and going through my code line-by-line didn't help (I missed the typo every time).

Calling the `check_for` function on my *initial* half-edge element pointers made this obvious; realizing that the memory address of both `h1` and `h2` was identical, and I noticed the typo as indicated above. Setting `h2 = h1->next();` resolved the issue.

