In this lesson, we will learn about using 4x4 transformation matrices to change the position, rotation, and scale of 3D objects. So far, we assumed that the geometry we rendered was always positioned where the model was initially created. We learned how to ray-trace spheres with the arbitrary center position. Though the position of a polygon meshes in the scene is defined by the position of the vertices making up the mesh, and in most cases, this is not where we want the object to be in the final scene. Thus we need to transform it. Imagine for example that you have modeled a tree but want to render an image of a forest. To create the forest you will use the model of the tree you created, duplicate this model a large number of times, and apply random transformations to these copies to make each tree's scale, position, and rotation unique. By doing so, you somehow can make the model look like it is unique, giving the illusion that the forest is made of many different tree models. Transformation more generally is useful to place any model into its final position in the rendered scene (position here is used in a general sense, i.e. it includes the concept of position, rotation, and scale). In CG, artists call this step **layout** or **set dressing** (layout may also include placement of cameras in the scene which are themselves transformed). It consists of taking objects as they were modeled (eventually taking multiple copies of the same model), and moving, rotating, and scaling them around.

![](/images/transformations/trees-transform.png?)

![Figure 1: from object to world space.](/images/transformations/cow-transform.png?)

When they are created in modeling software such as Maya or Blender, 3D models are generally centered around the world's origin. More often the base of the model also lies in the xz-plane (in Maya, or xy plane in 3DSMax). When they are in this position, we say that the model is defined in **object space**. If we change the size, rotation, and position of this object using a 4x4 transformation matrix, for example, we say the object is defined in **world space** and the matrix transforms the object from object to world space, is of course call the **object-to-world matrix** (in OpenGL and more generally real-time graphics APIs, this matrix is also known as the **model** matrix).

## Transforming Vertices

From a programming point of view, transforming the object from object space to world space is straightforward. All we need to do is loop over all the vertices of the mesh and transform them with the object-to-world matrix:

```
for (uint32_t i = 0; i < numVertices; ++i) { 
    const Vec3f & vertObjectSpace = vertices[i]; 
    Vec3f vertWorldSpace; 
    objectToWorld.multVecMatrix(vertObjectSpace, vertWorldSpace); 
    vertices[i] = vertWorldSpace; 
} 
```

The code above is written for clarity but it can be made faster. The object-to-world matrix can often be easily queried in 3D applications. For example in Maya you can select the object and use the Mel command `xform -q -ws -m` or `getAttr .worldMatrix`.

The object-to-world matrix is a property of any object in a scene, thus, we can define it as a member variable of the `Object` class in our program. The matrix will be passed to the constructor of the class (line 4).

```
class Object 
{ 
 public: 
    Object(const Matrix44f &o2w) : objectToWorld(o2w) {} 
    virtual ~Object() {} 
    virtual bool intersect(const Vec3f &, const Vec3f &, float &, uint32_t &, Vec2f &) const = 0; 
    virtual void getSurfaceProperties(const Vec3f &, const Vec3f &, const uint32_t &, const Vec2f &, Vec3f &, Vec2f &) const = 0; 
    Matrix44f objectToWorld; 
}; 
```

All objects will have access to this matrix since they are all derived from this base class (quadrics, polygon meshes, etc.). For example, here is how it works for the `TriangleMesh` class. The object-to-world matrix is passed to the constructor of the `TriangleMesh` class (line 6) which in turn passes it on to the constructor of the `Object` class (line 13). Finally, in the constructor of the triangle mesh class, we loop over all the vertices making the mesh and set the mesh vertices to the input vertices transformed by the object-to-world matrix (lines 19-22):

```
class TriangleMesh : public Object 
{ 
public: 
    // Build a triangle mesh from a face index array and a vertex index array
    TriangleMesh( 
        const Matrix44f &o2w, 
        const uint32_t nfaces, 
        const std::unique_ptr<uint32_t []> &faceIndex, 
        const std::unique_ptr<uint32_t []> &vertsIndex, 
        const std::unique_ptr<Vec3f []> &verts, 
        std::unique_ptr<Vec3f []> &normals, 
        std::unique_ptr<Vec2f []> &st) : 
        Object(o2w), 
        numTris(0) 
    { 
        ... 
        // allocate memory to store the position of the mesh vertices
        P = std::unique_ptr<Vec3f []>(new Vec3f[maxVertIndex]); 
        for (uint32_t i = 0; i < maxVertIndex; ++i) { 
            // transform object space vertex to world space position
            objectToWorld.multVecMatrix(verts[i], P[i]); 
        } 
        ... 
    } 
}; 
```

<details>
About the transformation pipeline.
![](/images/transformations/transform-pipeline.png?) 
We now have a full picture of the transformation pipeline. Objects are first transformed from object space to world space. If rasterization is used, objects are then transformed from world-to-camera space. Vertices are then projected onto the screen (using the perspective projection matrix) to screen space and then remapped to NDC (as part of the perspective projection matrix). Finally, points on the image plane in NDC coordinates are converted to their final raster or pixel coordinates. In ray tracing, we only care about the object-to-world transformation matrix. Rays are defined in world space and the ray-geometry intersection test occurs in world space (it can also take place in object space as explained later in this lesson though this is generally more a special case).
</details>

## Transforming Normals

Remember from the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-normals) that when we transform points and vectors by a 4x4 matrix, normals have to be transformed by the **transpose of the inverse of that object-to-camera matrix**. Thus, in addition to setting up the object-to-world matrix in the Object class constructor, we will also compute the world-to-object matrix (line 4), which is going to be needed to transform normals:

```
class Object 
{ 
 public: 
    Object(const Matrix44f &o2w) : objectToWorld(o2w), worldToObject(o2w.inverse()) {} 
    virtual ~Object() {} 
    virtual bool intersect(const Vec3f &, const Vec3f &, float &, uint32_t &, Vec2f &) const = 0; 
    virtual void getSurfaceProperties(const Vec3f &, const Vec3f &, const uint32_t &, const Vec2f &, Vec3f &, Vec2f &) const = 0; 
    Matrix44f objectToWorld, worldToObject; 
}; 
```

In the constructor of the `TriangleMesh`, we will now need to transform the normal by the transpose of the world-to-object matrix. We first compute the transpose matrix (line 27) and use it to transform all normals (line 33):

```
TriangleMesh( 
    const Matrix44f &o2w, 
    const uint32_t nfaces, 
    const std::unique_ptr<uint32_t []> &faceIndex, 
    const std::unique_ptr<uint32_t []> &vertsIndex, 
    const std::unique_ptr<Vec3f []> &verts, 
    std::unique_ptr<Vec3f []> &normals, 
    std::unique_ptr<Vec2f []> &st) : 
    Object(o2w), 
    numTris(0) 
{ 
    uint32_t k = 0, maxVertIndex = 0; 
    // find out how many triangles we need to create for this mesh
    for (uint32_t i = 0; i < nfaces; ++i) { 
        numTris += faceIndex[i] - 2; 
        for (uint32_t j = 0; j < faceIndex[i]; ++j) 
            if (vertsIndex[k + j] > maxVertIndex) 
                maxVertIndex = vertsIndex[k + j]; 
        k += faceIndex[i]; 
    } 
    maxVertIndex += 1; 
 
    // allocate memory to store the position of the mesh vertices
    P = std::unique_ptr<Vec3f []>(new Vec3f[maxVertIndex]); 
    N = std::unique_ptr<Vec3f []>(new Vec3f[maxVertIndex]); 
    texCoordinates = std::unique_ptr<Vec2f []>(new Vec2f[maxVertIndex]); 
    Matrix44f transformNormals = worldToObject.transpose(); 
    for (uint32_t i = 0; i < maxVertIndex; ++i) { 
        // transform vertex
        objectToWorld.multVecMatrix(verts[i], P[i]); 
 
        // transform normal
        transformNormals.multDirMatrix(normals[i], N[i]); 
        N[i].normalize(); 
 
        texCoordinates[i] = st[i]; 
    } 
 
    // allocate memory to store triangle indices
    trisIndex = std::unique_ptr<uint32_t []>(new uint32_t [numTris * 3]); 
    uint32_t l = 0; 
    // generate the triangle index array and set normals and st coordinates
    for (uint32_t i = 0, k = 0; i < nfaces; ++i) {  //for each  face 
        for (uint32_t j = 0; j < faceIndex[i] - 2; ++j) {  //for each triangle in the face 
            trisIndex[l] = vertsIndex[k]; 
            trisIndex[l + 1] = vertsIndex[k + j + 1]; 
            trisIndex[l + 2] = vertsIndex[k + j + 2]; 
            l += 3; 
        } 
        k += faceIndex[i]; 
    } 
} 
```

Note that texture coordinates don't need to be transformed. They live in their own 2D space and are not affected by the object's transformation. It is of course possible to transform texture coordinates but a 3x3 matrix, in this case, should be enough (to handle scale, rotation, and translation in 2D space). Though in practice this is rarely done. Texture coordinates are generally adjusted in the 3D application and the transformations are directly baked into the coordinates themselves.

If we put all the pieces together and render a new object (the famous Utah teapot, an iconic object in the world of computer graphics) we can produce the two following images:

![](/images/transformations/ray-tracing-transform-results.png?)

The image on the left uses face normals and the image on the right uses vertex normals. Note that the faceted look of the object in the left image disappeared in the right image. This is what vertex normals are used for, but don't worry too much about it, we will learn about smooth shading in the next lesson. While it is not obvious in these images that the teapot has been transformed, you can change the matrix in the program to the identity matrix and render the image again. You will more easily be able to spot the difference and the effect of the object-to-world matrix.

## A Special Use of the World-to-Object Matrix in Ray-Tracing

![Figure 2: the ray-sphere intersection technique only works when the sphere is centered at the origin. In a) because the sphere was moved away from the origin, the ray-sphere intersection test would return false (the ray would miss the sphere because it would intersect it as if it was centered at the origin). In b) we apply the world-to-object matrix from the sphere to the ray's origin and direction which transforms the ray as if the sphere was located at the origin. We can then safely apply the ray-sphere intersection routine.](/images/transformations/raysphereisect.png?)

In this part of the lesson, we will show one interesting use of the world-to-object matrix in ray tracing. In the lesson [A Minimal Ray-Tracer: Rendering Simple Shapes (Sphere, Cube, Disk, Plane, etc.)](/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-sphere-intersection) we studied two techniques to compute the ray-sphere intersection, a geometric and a parametric method. While it is possible in both methods to specify the center and the radius of the sphere we wish to render, let's imagine a situation in which we don't know how to ray-trace a sphere unless the sphere has radius 1 and is centered about the world coordinate system origin. How could we still change the sphere size, rotation, and position? The solution to this problem is simple. If the sphere's new scale, position, and rotation are defined by a 4x4 transformation matrix, then rather than transforming the sphere using this matrix, we will transform the ray instead of the sphere to the sphere object space, by transforming its position and direction using the sphere world-to-object matrix (the inverse of the sphere object-to-world matrix). We don't transform the sphere, we transform the ray by the sphere's matrix inverse. Imagine that we translate the sphere by 2 units in Y. Then instead of computing the intersection of the ray with the translated sphere, we keep the sphere at the origin, and move the ray position by -2 in Y (as shown in figure 2). This technique is quite common in ray tracing. It is sometimes more convenient to compute the intersection of a shape while the shape is in its object space. Thus rather than transforming the shape by the object-to-world matrix and testing the ray-shape intersection with the ray in world space, we transform the ray to the shape of object space and perform the ray-geometry test in object space instead.

```
bool intersect(const Vec3f &orig, const Vec3f &dir, float &t) const 
{ 
    Vec3f origObject, dirObject; 
    worldToObject.multVecMatrix(orig, origObject); 
    worldToObject.multDirMatrix(dir, dirObject); 
    float t0, t1;  //solutions for t if the ray intersects 
    // analytic solution
    float a = dirObject.dotProduct(dirObject); 
    float b = 2 * dirObject.dotProduct(origObject); 
    float c = origObject.dotProduct(origObject) - 1;  //radius = 1 and radius^2 = 1 
    if (!solveQuadratic(a, b, c, t0, t1)) return false; 
    if (t0 > t1) std::swap(t0, t1); 
 
    if (t0 < 0) { 
        t0 = t1;  //if t0 is negative, let's use t1 instead 
        if (t0 < 0) return false;  //both t0 and t1 are negative 
    } 
 
    t = t0; 
 
    return true; 
} 
```

The problem with this technique is that \(t\) the intersection distance, is not in world space anymore but in object space. The value computed by this method can't be compared to the value returned for instance by the `intersect()` method of the `TriangleMesh` class. To fix the problem, you need to compute the intersection point in object space, transform the hit point to world space, and then compute \(t\) by taking the distance between the ray origin and the hit point in world space. All these complications are worth it if computing the ray-geometry intersection is much more efficient in object space than in world space. We won't implement this method in our program. Mentioning the method is enough but you can give it a try as an exercise by adapting the code above.

## Source Code

As usual, the source code of the full program can be found in the last chapter of this lesson.

## Exercises

Render multiple copies of the teapot geometry using different transformation matrices.