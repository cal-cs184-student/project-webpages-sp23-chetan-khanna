# Project 3-1 Task 2

# BVH Construction Algorithm Walkthrough

The bounding volume hierarchy is constructed recursively, which is expected since we are constructing a tree.

To construct it, we first create an empty bounding box object.

We then iterate through all passed in primitives (denoted by the `start` and `end` iterators), expanding the BBox one at a time. We will always do this (leaf node or not), since a bounding box encloses all it's primitives regardless of the level it is at.

Since each primitive has it's own `BBox` and `BBox` has an `expand` method that accepts another `BBox`, expansion is easy to do. Note that since we were passed in an iterator that iterates through a vector of pointers to primitives, we need to "dereference" twice to get the primitive's `BBox`, which is achieved throguh `(*p)->get_bbox()`.

(Note: Inside this loop I also take the `BBox` of every centroid and add it to a running sum `allCentroids` (which was a `Vector3D` initialized to zero). I do this in case I need to find a split point for this bounding box, which I explain later.)

After looping through all primitives and expanding the `BBox`, I build a node (calling `new BVHNode(bbox)`) with the bounding box.

(Notice that this is not taking a *pointer* to the `bbox`, but rather the `bbox` itself, which means that the BBox is not on the stack but rather directly "inside" the `BVHNode` on the heap. Thus even after we exit out of this function, the bounding box continues to exist.)

If the number of primitives is more than the number of primitives we allow in a leaf node, then `thisNode` cannot be a leaf node, so we recurse down. In order to do this, we must find a split point in our bounding box so that the child nodes split the primitives in some way.

I chose my split point by taking the longest axis of the current bounding box, then find the mean point on that axis (based on the centroid’s of all the primitives).

I used the `Vector3D` operations directly for speed, performing `allCentroids / double(numSoFar)` to get the `meanSplitPoint` for every axis, and then choosing the longest axis by using `bbox.extent`. Based on the longest entry of `bbox.extent`, I set `axisToSplit` appropriately.

After this, I needed to rearrange the primitives (delimited by `start` and `end`) so that the left and right child nodes had the correct primitives in them based on my split point.

For this, I first created an lambda function `splitPointFunction` which returned `false` if the input primitive's value in `axisToSplit` was less than the `meanSplitPoint[axisToSplit]` (and thus should go in the left BVH node) or `true` for the opposite case.

I then called `std::partition`, which rearranged `start` and `end` based on my `splitPointFunction` and returned an iterator `secondGroupIterator` to the vector of pointers to primitives for the right BVH node. I called `construct_bvh` recursively then with the appropriate arguments for each child node (left: `start, secondGroupIterator, max_leaf_size` and right: `secondGroupIterator, end, max_leaf_size`), setting the results to the corresponding children of `thisNode`, and then returning `thisNode` (as per normal recursion).

Note that between the partitioning and recursive calls to `construct_bvh` I also checked if any of the partitioned groups were of size zero; if they were, we would recurse infinitely, since our algorithm isn't actually constructing smaller BVHs (we're not splitting the problem recursively). In such a scenario I simply force the left and ride nodes to split with an equal number of primitives regardless of their location. This is not necessarily the most efficient strategy recursively; an alternative would be to simply make one of the nodes have *one* primitive. However, it nevertheless works well.

## Images with Normal Shading for a Few Large Files

![Part2Beetle.png](Project%203-1%20Task%202.assets/Part2Beetle.png)

![Part2CBLucy.png](Project%203-1%20Task%202.assets/Part2CBLucy.png)

![Part2MaxPlanck.png](Project%203-1%20Task%202.assets/Part2MaxPlanck.png)

## Comparison of rendering times on a few scenes with moderately complex geometries

| **Image/Scene** | Without BVH rendering time | Without BVH average intersection tests per ray | With BVH rendering time | With BVH average intersection tests per ray |
| --------------- | -------------------------- | ---------------------------------------------- | ----------------------- | ------------------------------------------- |
| Cow             | 3.5819s                    | 1095.896793                                    | 0.0362s                 | 4.051131                                    |
| Lucy            | *Unworkable*               | *Unworkable*                                   | 0.0373s                 | 4.008021                                    |
| Wall-e          | 351.0641s                  | 156964.646935                                  | 0.0494s                 | 5.893357                                    |

Naturally, due to the logarithmic asymptotic runtime (based on triangles), the difference in performance between the naïve version and the BVH version becomes even larger, as is demonstrated by the progressively larger benefit as we jump to more complex geometries, from cow to Lucy.

