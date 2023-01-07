## Ray-Tracing a Polygon Mesh

As we suggested before, ray-tracing a polygon mesh which has been triangulated is simple. We already have a routine to compute the intersection of rays and triangles. Therefore, to test if a ray intersects a polygon mesh, we need to loop over all the triangles in the mesh and test each individual triangle against the ray. However, a ray may intersect more than one triangle from the mesh. Therefore, we must keep track of the nearest intersection distance as we iterate over the triangles. This problem is similar to the one described in the lesson [A Minimal Ray-Tracer](/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/minimal-ray-tracer-rendering-spheres) where we had to keep track of the nearest intersected object. We will use the same technique here. In pseudo-code, the intersection routine looks like this (check the last chapter for the C++ implementation):

```
// Test if the ray intersects this triangle mesh
bool intersect(const Vec3f &orig, const Vec3f &dir, float &tNear, uint32_t &triIndex, Vec2f &uv) const 
{ 
    uint32_t j = 0; 
    bool isect = false; 
    for (uint32_t i = 0; i < numTris; ++i) { 
        const Vec3f &v0 = P[trisIndex[j]]; 
        const Vec3f &v1 = P[trisIndex[j + 1]]; 
        const Vec3f &v2 = P[trisIndex[j + 2]]; 
        float t = kInfinity, u, v; 
        if (rayTriangleIntersect(orig, dir, v0, v1, v2, t, u, v) && t < tNear) { 
          tNear = t; 
          uv.x = u; 
          uv.y = v; 
          triIndex = i; 
          isect |= true; 
        } 
        j += 3; 
    } 
 
    return isect; 
}
```

We need the three vertices making up each triangle in the mesh (lines 7-9). We then pass these three points to the ray-triangle intersection routine, which returns true if the ray intersects the triangle and false otherwise (we will use the Möller-Trumbore ray-triangle algorithm studied in the previous lesson). In the case of an intersection, the variable \(tNear\) is set with the intersection distance (the distance from the ray origin to the intersection point on the triangle) and the barycentric coordinates of the hit point (u & v). We also keep track of the triangle index that the ray intersected and the barycentric coordinates of the intersected point on the triangle (lines 13-15). This will be needed later to calculate the hit point's normal and texture coordinates.

## Creating a Poly Sphere

We need a polygon mesh to test our code. One solution is to use an existing model, but this solution requires choosing a file format and writing code to read and parse models exported to this format. Another solution consists of creating some simple geometry on the fly procedurally. In this chapter, we will use this option. In the next chapter, we will render a polygon mesh defined in an external file. Building a polygon object procedurally is also an opportunity to put into practice what we learned in the previous chapter. We will generate a sphere as a 3D polygon mesh using the following code:

```
TriangleMesh* generatePolyShphere(float rad, uint32_t divs) 
{ 
    // generate points                                                                                                                                                                                      
    uint32_t numVertices = (divs - 1) * divs + 2; 
    std::unique_ptr<vec3f []=""> P(new Vec3f[numVertices]); 
    std::unique_ptr<vec3f []=""> N(new Vec3f[numVertices]); 
    std::unique_ptr<vec2f []=""> st(new Vec2f[numVertices]); 
 
    float u = -M_PI_2; 
    float v = -M_PI; 
    float du = M_PI / divs; 
    float dv = 2 * M_PI / divs; 
 
    P[0] = N[0] = Vec3f(0, -rad, 0); 
    uint32_t k = 1; 
    for (uint32_t i = 0; i < divs - 1; i++) { 
        u += du; 
        v = -M_PI; 
        for (uint32_t j = 0; j < divs; j++) { 
            float x = rad * cos(u) * cos(v); 
            float y = rad * sin(u); 
            float z = rad * cos(u) * sin(v) ; 
            P[k] = N[k] = Vec3f(x, y, z); 
            st[k].x = u / M_PI + 0.5; 
            st[k].y = v * 0.5 / M_PI + 0.5; 
            v += dv, k++; 
        } 
    } 
    P[k] = N[k] = Vec3f(0, rad, 0); 
 
    uint32_t npolys = divs * divs; 
    std::unique_ptr<uint32_t []=""> faceIndex(new uint32_t[npolys]); 
    std::unique_ptr<uint32_t []=""> vertsIndex(new uint32_t[(6 + (divs - 1) * 4) * divs]); 
 
    // create the connectivity lists                                                                                                                                                                        
    uint32_t vid = 1, numV = 0, l = 0; 
    k = 0; 
    for (uint32_t i = 0; i < divs; i++) { 
        for (uint32_t j = 0; j < divs; j++) { 
            if (i == 0) { 
                faceIndex[k++] = 3; 
                vertsIndex[l] = 0; 
                vertsIndex[l + 1] = j + vid; 
                vertsIndex[l + 2] = (j == (divs - 1)) ? vid : j + vid + 1; 
                l += 3; 
            } 
            else if (i == (divs - 1)) { 
                faceIndex[k++] = 3; 
                vertsIndex[l] = j + vid + 1 - divs; 
                vertsIndex[l + 1] = vid + 1; 
                vertsIndex[l + 2] = (j == (divs - 1)) ? vid + 1 - divs : j + vid + 2 - divs; 
                l += 3; 
            } 
            else { 
                faceIndex[k++] = 4; 
                vertsIndex[l] = j + vid + 1 - divs; 
                vertsIndex[l + 1] = j + vid + 1; 
                vertsIndex[l + 2] = (j == (divs - 1)) ? vid + 1 : j + vid + 2; 
                vertsIndex[l + 3] = (j == (divs - 1)) ? vid + 1 - divs : j + vid + 2 - divs; 
                l += 4; 
            } 
            numV++; 
        } 
        vid = numV; 
    } 
 
    return new TriangleMesh(npolys, faceIndex, vertsIndex, P, N, st); 
}
```

![Figure 1: a sphere with 5 divisions. Left: a face is created by connecting vertices. Right: the sphere's top and bottom rows are triangle fans (all the triangles share one vertex at the center). The other faces are quads.](/images/ray-triangle-mesh/polysphere.png?)

The number of divisions represents the number of stacks and slices along and around the y-axis. For example, the sphere in figure 1 uses five divisions. We start by creating all the vertices. This example's total number of points is 22 (2 for the top and bottom of the spheres plus 5 times 4 rings of points). We create the position for these vertices using trigonometric functions (lines 20-22). To complete the mesh description, we need to provide information about the faces' connectivity. The faces at the top and bottom of the spheres are triangles (the top and bottom cap make what we call a triangle fan: all the triangles share one central vertex). They are defined by three vertices (lines 42-44 and 49-51). The others are quads (line xx). Finally, the faces are created by connecting vertices in a precise order. Figure 1 shows the vertex indexes for the sixth face in the sphere (lines 56-59). Note that we also generate vertex normals and texture coordinates in the code. Normals are simple to compute. Since the sphere is centered around the world origin, the normal at the vertex is simply the normalized vertex position. For the texture coordinates, all we need to need to do is normalize the parameters u (the horizontal angle \(\phi\)) and v (the vertical angle \(\theta\)), which lie in the range \([-\pi,\pi]\) and \([-\pi/2,\pi/2]\) respectively (lines 24-25). The <span class="code-inline">intersect()</span> method implements the Möller-Trumbore algorithm for the ray-triangle intersection test. Images in figure 2 were obtained by increasing the number of divisions for the polygon sphere.

The source code can be found on [GitHub](https://github.com/scratchapixel/code).

## Performances

![Figure 2: render time (vertical axis, in seconds) increases linearly with the number of triangles in the scene (horizontal axis).](/images/ray-triangle-mesh/rtlinear.png?)

![Figure 3: increasing the number of subdivisions.](/images/ray-triangle-mesh/ray-tri-spheres.gif?)

Suppose we regularly increase the number of faces making the sphere and measure the time it takes to render a frame. In that case, we can see in figure 3 that the render time increases linearly with the number of triangles in the scene (there is a linear dependence between the render time and the number of triangles in the scene). We can already realize from the numbers we get for a simple scene that ray-tracing is "slow" (and processors today are incredibly faster than in the early days of ray-tracing). A scene containing approximately 200 polygons takes around 2.5 seconds to render on a computer equipped with a 2.5 GHz processor.

![Figure 4: if a ray doesn't intersect the mesh bounding box, we don't need to test if the ray intersects any of the mesh triangles.](/images/ray-triangle-mesh/ray-tri-bbox.png?)

Can this problem be solved? Ray-tracing will always be slower than rasterization, but things can be improved quite significantly with the help of acceleration structures. The problem with ray tracing is that each ray needs to be tested against every single triangle in the scene. No matter how small the object is in the scene (see the next chapter), the time it will take to produce a frame is constant. It only depends on how many triangles the scene contains. The time it takes to render a pixel is constant, whether this pixel is in the top-left corner of the frame or right in the middle of it. The idea behind acceleration structures is simple. For example, we could first test if a ray intersects the object's bounding box. If it doesn't, then we know that the ray can't hit the object. If it does, we can still test if the ray intersects any of the mesh triangles (figure 4). This straightforward test can already save a lot of time. For example, all the pixels in the corners of the frame are unlikely to intersect the mesh's bounding box. Acceleration structures are more complex than bounding boxes but are based on the same idea. They can be used to divide the space the objects fill into simple volumes that are fast to ray-trace. If rays intersect these sub-volumes, we can go deeper into the structure and test the ray against smaller sub-volumes until we eventually hit a sub-volume containing the mesh original triangle. At this point of the process, we test the ray against the triangles contained in the sub-volume. Lessons on acceleration structures can be found in the Advanced Ray-Tracing section.

```
int main(int argc, char **argv) 
{ 
    // setting up options
    Options options; 
    options.cameraToWorld[3][2] = 4; 
 
    for (uint32_t i = 0; i < 10; ++i) { 
        int divs = 5 + i; 
        // creating the scene (adding objects and lights)
        std::vector<std::unique_ptr<Object>> objects; 
        TriangleMesh *mesh = generatePolyShphere(2, divs); 
        objects.push_back(std::unique_ptr<Object>(mesh)); 
 
        // finally, render
        auto timeStart = std::chrono::high_resolution_clock::now(); 
        render(options, objects); 
        auto timeEnd = std::chrono::high_resolution_clock::now(); 
        auto passedTime = std::chrono::duration<double, std::milli>(timeEnd - timeStart).count(); 
        std::cerr << mesh->numTris << " " << passedTime << std::endl; 
    } 
 
    return 0; 
}
```

## Conclusion

In this lesson, we gave some information about the polygon geometry representation and showed how this geometry could be rendered with a ray tracer. The polygons (if they are convex) can be converted to triangles, and an efficient algorithm (the Möller-Trumbore method, for instance) can then be used to compute the intersection of rays with these triangles. Converting all the different geometry representations into triangles is easier than supporting a fast and robust ray-geometry intersection method for each one of these representations. It requires only support for a ray-triangle intersection routine which can be highly optimized. Finally, in this chapter, we showed one of the essential properties of the ray-tracing algorithm: the computational time is linearly dependent on the number of objects or triangles in the scene. Despite the progress made with processors, using a naive implementation of ray tracing to render reasonably complex scenes becomes quickly unpractical. Hopefully, the cost of testing each object in the scene with each ray can be significantly reduced if we use some acceleration techniques.

## What's Next?

In the next chapter, we will render the image we produced in the lesson on rasterization using ray tracing. We will load the object geometry from the disk, pass the data to the `TriangleMesh` constructor, generate a triangle mesh, loop over all the pixels in the image, generate primary rays, and test each one of the rays against each triangle in the mesh.