# CS 184 Project 2 Task 5

- Briefly explain how you implemented the edge split operation and describe any interesting implementation / debugging tricks you have used.
- Show screenshots of a mesh before and after some edge splits.
- Show screenshots of a mesh before and after a combination of both edge splits and edge flips.
- Write about your eventful debugging journey, if you have experienced one.
- If you have implemented support for boundary edges, show screenshots of your implementation properly handling split operations on boundary edges.

# Brief Explanation: Implementation of Edge Split Operation

Edge split is fundamentally similar to edge flip in that it is about reassigning pointers—especially those of half-edges’. However, it is more complicated in that it requires the creation of elements. (In my first implementation, I also deleted some elements but I removed this later for bookkeeping and performance reasons.)

I first show a diagram to show the edge split operation.

### Diagram

The diagram I drew to demonstrate the edge flip is as follows:

**Before split**

![Image.jpeg](CS%20184%20Project%202%20Task%205.assets/Image.jpeg)

**After split**

![Image.jpeg](CS%20184%20Project%202%20Task%205.assets/Image%20(2).jpeg)

- **Purple:** new half-edges created by the split; note `h3` and `h0` simply have their pointers reassigned to them (rather than being altogether deleted and new edges created in their place)
- **Green:** new faces created; note `f0` and `f1` are simply assigned to new half-edges and are now smaller (not deleted and new ones created in their place)
- **Turquoise:** new edges created by the split; note `e0` is simply assigned a new half-edge
- In **brown**, only `v4` is a new created vertex, positioned at the midpoint of `e0`.

## More in code

The *actual* coding portion is very similar to task 4, with the exception of the different pointer assignments as in the diagram above and the creation of new elements as mentioned in the paragraph above.

I thus do not go through the code step-by-step, but rather mention important differences.

I firstly set up my pointers to the local neighborhood of `e0` in identical fashion to the flip edge operation.

I also calculate the midpoint of `e0` in order to find the midpoint. I store this for now; the position of the new vertex I create for the split will be given this position.

I then construct the new elements as in the color-coded “after split” diagram above. In total we have six new half-edges, three new edges, one new vertex, and two new faces. I simply call `newHalfedge()`, `newEdge()`, etc. I assign these to new variables as in the diagram above (e.g. `HalfedgeIter h12 = newHalfedge()`. **Crucially, these elements are initialized but not actually pointing to anything yet; they're not connected to the mesh.** I will connect them to the mesh as I reassign pointers in the next step.

The next phase of my code follows logic analogous to task 4; I thus skip a detailed explanation here. I use `setNeighbors` to set the half-edges up, and then I assign the other element types afterwards. (This obviously includes the new elements, which means that these new elements now get connected to the mesh). Note how existing elements such as `h0` are reassigned to `v4` (so their source vertex is now the split midpoint); I don't delete these elements and re-create them, I just reassign their pointers. This helps with bookkeeping, minimizing bugs, and also reduces the amount of new elements created and deleted, thus helping performance.

I then do two final things. The first is to actually position the new vertex created to be the at the midpoint (since we wanted to split at the midpoint). This means I do `v4->position = toMidpoint`, i.e. assign it's position to be the midpoint of the split edge calculated earlier.

I also modify the `isNew` property (which is used during Loop Subdivision in Task 6) of several elements. Specifically, edges that existed before, but have now been split should still be marked as old (`isNew = false`), even if one of those edges was just *constructed.* In other words, if an edge was formerly part of a bigger "un-split” edge, it should be marked old. This situation applies to `e7` in the diagram above, where we set `e7->isNew = false` even though it was just created. *Truly* new edges created by the split are set to be new (`e5` and `e8` in the diagram), and so is the new vertex (`v4`).

The use and purpose of `isNew` is detailed in Task 6.

# Screenshots of a mesh before and after some edge splits

![Image.png](CS%20184%20Project%202%20Task%205.assets/Image.png)

# Screenshots of a mesh before and after a combination of edge flips and edge splits

![Image.png](CS%20184%20Project%202%20Task%205.assets/Image%20(2).png)

![Image.png](CS%20184%20Project%202%20Task%205.assets/Image%20(3).png)

# No Debugging Needed!

My program worked perfectly on the first go. I thus did not have an “eventful debugging journey.”

I credit this to drawing a diagram carefully and then also going through each line of code as I typed it.

