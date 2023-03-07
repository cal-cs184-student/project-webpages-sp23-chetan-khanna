# CS 184 Project 2 Task 3

# Brief Explanation: Implementation of Area-Weighted Vertex Normals

We need area-weighted normal vector across much of shading; for example, Blinn-Phong Specular Shading’s intensity depends on view direction. Indeed, even Lambert’s Cosine Law, which we use to determine the amount of reflected light is defined by $$n \cdot l =  \text{cos } \theta$$.

To get the area-weighted normal vector at a vertex, I initialize a `Vector3D areaWeightedNormals` and iterate through all the faces incident at that vertex (adding the area-weighted normal vector for that face to `areaWeightedNormals`). Finally, I normalize the `areaWeightNormals` vector so it is of length ~1 and then return it.

#### Traversing faces

In order to traverse faces, I used the `twin` pointer of a half-edge in a `do-while` loop. Specifically, I started at the half-edge pointed to by this vertex's `halfedge` pointer. This half-edge has (among other things) a face associated with it, for which I got the area-weighted vertex normal (explained below). Then, to iterate to the next face that this vertex has, I called the `twin` (which moved us to the next neighboring face connected to the vertex) and then called `next` (in order to ensure that our vertex remains this vertex, which is important when we try to jump to the next face again).

Note how the termination condition is when the current half-edge we are at is the same as the `halfedge` pointed to by this vertex's `halfedge` pointer: this means we terminate upon reaching the same half-edge and therefore face.

#### Getting an area-weighted vertex normal for once face

To get an area-weighted vertex normal for one face, I use the half-edge data structure to get two neighboring vertices in one face, take their cross product (to get a vector normal to the face that is of length equal to the area of the face, and thus *area-weighted*).

I first take the “root” vertex `A` position, which is given by `Vector3D A = thisHalfedge->vertex()->position`. This "root vertex" will be used as the "common vertex" of the two edges used in the cross product. I get the other vertex `B` of this face by `Vector3D B = thisHalfedge->next()->vertex()->position`, and then construct `Vector3D AB = B - A` (we need the exact vectors in order to perform a cross product properly). I get the vertex `C` on this face by doing `Vector3D C = inputHalfedge->next()->next()->vertex()->position` and then construct `Vector3D AC = C - A`.

**Side note:** notice how I never had to use `twin` because I am not going to a different face; to get the next vertex, I simply used `next()`, and then `vertex()` to get the half-edge’s source vertex.

I then call `cross(AB, AC)` in order to get a normal vector with length equal to the area of the face.

# Screenshots of teapot with and without vertex normals

### Set 1 of screenshots

#### Without vertex normals implemented

![without normals.png](CS%20184%20Project%202%20Task%203.assets/without%20normals.png)

#### With vertex normals implemented

![with normals.png](CS%20184%20Project%202%20Task%203.assets/with%20normals.png)

### Set 2 of screenshots

#### Without vertex normals implemented

![Image.png](CS%20184%20Project%202%20Task%203.assets/Image.png)

#### With vertex normals implemented

![Image.png](CS%20184%20Project%202%20Task%203.assets/Image%20(2).png)

