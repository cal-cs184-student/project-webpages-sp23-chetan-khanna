# CS 184 Project 2 Task 6

# Brief Explanation: Implementation of Loop Subdivision and Implementation/Debugging Tricks

Since the positions of new vertices as defined by Loop Subdivision is done based on their original positions (i.e. *before* the subdivision) (which makes sense—otherwise we'd have different amounts of “subdivision" depending on how much subdivision was done in the local neighborhood, which is nonsensical), I first calculate the positions of new and old vertices based before performing any actual edge splitting/subdivision.

This is better for performance; if we subdivided first, then we would have to traverse more half-edges to get to the original edges since we have more triangles (e.g. for example, an "original” neighboring vertex normally given by `thisHalfedge->next()->vertex()` may now be given by `thisHalfedge->next()->twin()->next()->vertex()`, reducing performance).

Thus, the first two sections of this task are: **(a) calculating the positions of "old**” **vertices** (i.e. existing vertices) and **(b)** **calculating the positions of** “**new" vertices** (i.e. vertices that will be created by splitting edges during subdivision).

Note that we check for boundary edges and ignore them in case we come across one.

#### Part (a): calculating the positions of "old” vertices

We iterate through every vertex in the mesh (no subdivision has happened yet so all of them are “old”), and use the provided Loop Subdivision Rule `(1 - n * u) * original_position + u * original_neighbor_position_sum.` In order to calculate `n`, I simply check `thisVertex->degree()` (we are not dealing with boundary edge cases; if we were, then in those cases `n = thisVertex()->degree() + 1`). I then manually calculate `u` with the given rule based on `n`.

To sum all neighbors, I initialize a `Vector3D(0.0, 0.0, 0.0)` and add the positions of all neighboring vertices iteratively to it. I traverse the neighboring vertices using a `do-while` loop as in standard fashion (see Task 3). *Side note: note how the next `halfedge` in the `do-while` loop is given by `currentHalfedge->twin()->next()`—this is because we need to move to the next face (`twin()`) and then come to the equivalent half-edge for this next face (`next()`). The next iteration itself will take care of calling `twin()` again (to get the half-edge with the opposite source vertex) and then adding `vertex()->position()` to the vector.*

We perform the calculation and store it in `vertex->newPosition`. We do **not** store it in `vertex->position` yet since we don't want to update this vertex's position yet! This is because this vertex's *original* position still may be used by neighboring vertices when we're calculating *their* new positions.

We also set `thisVertex->isNew() = false` since these vertices already exist and are thus old; they've not been created by this iteration of subdivision, of course!

#### Part (b): calculating the positions of new vertices

We calculate the positions of the new vertices that *will* be created by subdivision using the given Loop Subdivision rule `3/8 * (A + B) + 1/8 * (C + D)`, where $$A, B$$ are the current edges vertices, and $$C, D$$ are the "non-included“ vertices of the faces of either side of this edge (in other words, `edge->halfedge()->next()->next()->vertex()`, for both half-edges of this edge).

($$A, B$$ are therefore given by `thisEdge->halfedge()->vertex()->position` and `thisEdge->halfedge()->twin()->vertex()->position` respectively. $$C, D$$ are therefore given by `thisEdge->halfedge()->next()->next()->vertex()->position` and `thisEdge->halfedge()->twin()->next()->next()->vertex()->position` respectively.)

I store the calculated position in the *edge*’*s* `newPosition` attribute because the vertex hasn't been created yet, and since this position is for the vertex created by splitting this edge, this is a natural place to store this data. When this edge is split, I will simply take this `newPosition` and set it to the create vertex's `newPosition`.

Since we haven't subdivided in this iteration of Loop Subdivision yet, every edge is an old edge. I therefore set `thisEdge->isNew = false` here. This should *generally* be `false` anyways, but it's possible that this edge's `isNew` was set to true by `splitEdge` in a previous iteration of Loop Subdivision, so this is to cover that case.

#### Part (c): Subdivision Part 1—Splitting Edges, Subdivison Part 2—Flipping Edges

We now split every edge in the mesh in any order by calling `splitEdge`.

Since we only want to split “truly new” edges—and *not* any edges that were formerly part of a bigger, unsplit edge, we check that both vertices of the current edge are **old.** A new edge *or* an edge that was formerly part of a bigger edge will have (at least) one new vertex. Both vertices of a truly old edge will be old*. (Note that `splitEdge` sets the newly created vertex to `isNew`, so this is not handled directly in the code for this task—we just perform the check).

(Side note: this prevents an infinite loop; otherwise the creation of new edges in `splitEdge` would mean we never end splitting edges! It also means we could infinitely splitting select edges (those that were formerly part of a bigger edge) forever.)

We set the created vertex’s `newPosition` from `thisEdge->newPosition` as explained in part (b). We could set this vertex’s `position` directly but do not yet for code cleanliness.

*Let’s call this truly old Edge `thisOldEdge`. Recall that an edge is split at it's midpoint; it's therefore not possible for the split of a neighboring edge to cause any of `thisOldEdge`’s vertices to become new.

The second part of subdivision is flipping truly new edges that connect an old and a new vertex. This is necessary to “equalize” the mesh—if we did not, we wouldn’t get an *even* subdivsion. We check if an edge `isNew` and that exactly one of it's edges is new and the other is old. If it is, we call `flipEdge`.

#### Part (d): Update all Vertex Positions

We’ve already done the hard work of calculating each vertex’s new position in parts (a) and (b). We stored each vertex’s new position in `thisVertex->newPosition`.

We thus simply loop over all the vertices in the mesh (except for boundary ones, as usual) and set each vertex’s `newPosition` to it's `position`.

## Debugging Tricks

There were no specific *tricks* I used.

One of my issues was in Part (c), where I would end in an infinite loop. I suspected that this `splitEdge` was the issue from the outset, so I commented `splitEdge` to see if it caused it, and indeed the infinite loop stopped. I inspected `splitEdge` line-by-line (using my diagram), and found it to be correct; I thus looked at my function again. I noticed that I had forgotten to check if both my vertices were old (and I was just checking `isNew`)—it was thus splitting edges that were previously part of a bigger "un-split” edge infinitely (the `isNew` check prevented it from splitting the totally new edges, however). After adding this, this part worked well.

Another issue was that I was not using `float`/`double` values everywhere. I noticed this because some vertex positions became `(0,0,0)` or `(1,1,1)` immediately. Upon inspection, I noticed this was an issue in part (a) (where I was calculating the old vertex positions), where my fractions were casted to `ints`.

I also noticed that subdivision often caused some points to “implode” into the mesh. I checked which points these were, and realized these were all the old vertices. This gave me a clue that my part (a) was not returning the right positions. I wasn’t able to solve this for a while (I checked they were being assigned new positions correctly at the end of subdivision, and re-verified my flip and split edges to no avail). Eventually, using the debugger, I realized that I was traversing vertices of degree $$\text{n}$$ but only adding the values of $$\text{n - 1}$$ neighboring vertices. I realized that this was because I was calling calling `next()` one too many times in the `do-while` loop where I was summing the positions of neighboring vertices. Once I fixed this, Loop Subdivision worked correctly.

# Behavior post subdivision of sharp corners and edges

![Image.png](CS%20184%20Project%202%20Task%206.assets/Image.png)

![Image.png](CS%20184%20Project%202%20Task%206.assets/Image%20(2).png)

Sharp corners and edges become a lot *less* sharp after Loop Subdivision. In other words, we lose corners and edges. This is expected—the vertices get "pulled in" since they move towards their neighbors, which are "backwards" if the vertex itself is defined as the most "forward" point.

At a high level, splitting edges on one side of a vertex will increase the number of neighbors on that one side, thus pulling the vertex, after subdivision, to the side where we’ve done a lot of splits.

Therefore, if perform a lot of splits (preferably uniformly) around a vertex (equivalently, for an edge: on both sides of the edge), the neighbors will move after subdivision by a proportionally “smaller” amount (since their neighbors—and their neighbor’s neighbors—are that much closer), so that the new position of that vertex won’t be “pulled in” as much.

(Side note: the new vertices created also matter for later iterations of Loop Subdivision. In these cases, the new vertices created are also closer to the corner vertex/edge we are trying to preserve, and thus we have neighbors that are somewhat closer to the vertex itself for the next iteration of Loop Subdivision, thus preserving it’s position more.)

# Pre-processing the cube for symmetrical subdivision

### Showing Asymmetrical Subdivision of Cube

The following screenshot shows how the corners of the cube are not symmetrically positioned (nor symmetrically “sharp”. Note the “deformed orange” look.

![Screenshot 2023-03-05 at 23.54.44.png](CS%20184%20Project%202%20Task%206.assets/Screenshot%202023-03-05%20at%2023.54.44.png)

#### Explanation of why they occur and pre-processing for alleviation of these effects

Notice the edges of our cube before any subdivision occurs. In particular, note that there is an edge that splits each face, and that

Notice that on every face of the cube, there is an edge from the top left corner to the bottom right corner. However, there is no edge from the top right corner to the bottom left corner. Following the logic detailed in the section above (behavior of sharp corners and edges post-subdivision), this means that the top left vertices and bottom right vertices of each face will be preserved better as subdivision occurs.

In order to fix this, we can make the edges of each face symmetric. We split the existing edge that goes across each face, which creates a symmetrical face (an “X” pattern for every face). Again following the logic from above, this means that each vertex of the face gets “equal weight” and thus the vertices’ “sharpness” is preserved equally as Loop Subdivision takes place.

