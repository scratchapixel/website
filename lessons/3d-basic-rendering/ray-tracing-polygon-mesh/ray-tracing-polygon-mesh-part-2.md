Let's see how we can now reproduce the image we rendered in the lesson on rasterization. We know how to load a disk geometry file and render this object using ray tracing. Let's put these two techniques together in our program. In theory, if all works fine (mathematics never lies), we should get two perfectly identical images. Though if what we learned and said about ray tracing is true, ray tracing should take more time than rasterization. Let's validate these assumptions.

First, we will integrate the function to read the geometry file described in the lesson [Polygon Mesh](/lessons/3d-basic-rendering/introduction-polygon-mesh/polygon-mesh-file-formats) into our program's code. All the data read from the file (number of faces, the face and vertex arrays, the point, normal, and st coordinates arrays) are passed to the `TriangleMesh` constructor (line 5).

```
class TriangleMesh : public Object 
{ 
public: 
    // Build a triangle mesh from a face index array and a vertex index array
    TriangleMesh( 
        const uint32_t nfaces, 
        const std::unique_ptr<uint32_t []> &faceIndex, 
        const std::unique_ptr<uint32_t []> &vertsIndex, 
        const std::unique_ptr<Vec3f []> &verts, 
        std::unique_ptr<Vec3f []> &normals, 
        std::unique_ptr<Vec2f []> &st) : 
        numTris(0) 
    { 
        ... 
    } 
}; 
 
TriangleMesh* loadPolyMeshFromFile(const char *file) 
{ 
    std::ifstream ifs; 
    try { 
        ifs.open(file); 
        if (ifs.fail()) throw; 
        std::stringstream ss; 
        ss << ifs.rdbuf(); 
        uint32_t numFaces; 
        ss >> numFaces; 
        std::unique_ptr<uint32_t []> faceIndex(new uint32_t[numFaces]); 
        uint32_t vertsIndexArraySize = 0; 
        // reading face index array
        for (uint32_t i = 0; i < numFaces; ++i) { 
            ss >> faceIndex[i]; 
            vertsIndexArraySize += faceIndex[i]; 
        } 
        std::unique_ptr<uint32_t []> vertsIndex(new uint32_t[vertsIndexArraySize]); 
        uint32_t vertsArraySize = 0; 
        // reading vertex index array
        for (uint32_t i = 0; i < vertsIndexArraySize; ++i) { 
            ss >> vertsIndex[i]; 
            if (vertsIndex[i] > vertsArraySize) vertsArraySize = vertsIndex[i]; 
        } 
        vertsArraySize += 1; 
        // reading vertices
        std::unique_ptr<Vec3f []> verts(new Vec3f[vertsArraySize]); 
        for (uint32_t i = 0; i < vertsArraySize; ++i) { 
            ss >> verts[i].x >> verts[i].y >> verts[i].z; 
        } 
        // reading normals
        std::unique_ptr<Vec3f []> normals(new Vec3f[vertsIndexArraySize]); 
        for (uint32_t i = 0; i < vertsIndexArraySize; ++i) { 
            ss >> normals[i].x >> normals[i].y >> normals[i].z; 
        } 
        // reading st coordinates
        std::unique_ptr<Vec2f []> st(new Vec2f[vertsIndexArraySize]); 
        for (uint32_t i = 0; i < vertsIndexArraySize; ++i) { 
            ss >> st[i].x >> st[i].y; 
        } 
 
        return new TriangleMesh(numFaces, faceIndex, vertsIndex, verts, normals, st); 
    } 
    catch (...) { 
        ifs.close(); 
    } 
    ifs.close(); 
 
    return nullptr; 
} 
 
int main(int argc, char **argv) 
{ 
    ... 
    std::vector<std::unique_ptr<Object>> objects; 
    TriangleMesh *mesh = loadPolyMeshFromFile("./cow.geo"); 
    if (mesh != nullptr) objects.push_back(std::unique_ptr<Object>(mesh)); 
 
    // finally, render
    render(options, objects, 0); 
 
    return 0; 
}
```

The function `readGeometryFile()` returns a pointer to a new dynamically allocated instance of the `TriangleMesh` class. This instance holds the geometry data of the object we just read (the geometry has been triangulated at this point if it contained faces with more than 3 vertices). This instance is added to the object list and then passed on to the `render()` function (line 3).

```
void render( 
    const Options &options, 
    const std::vector<std::unique_ptr<Object>> &objects, 
    const uint32_t &frame) 
{ 
    ... 
    for (uint32_t j = 0; j < options.height; ++j) { 
        for (uint32_t i = 0; i < options.width; ++i) { 
            ... 
            *(pix++) = castRay(orig, dir, objects, options); 
        } 
    } 
    ... 
}
```

The rest of the program is conventional for a ray tracer. We loop over each pixel in the image, generate a primary ray (whose direction and origin are transformed by the camera-to-world 4x4 transformation matrix), and test if this primary intersects any of the objects in the scene. In the `castRay()`, we call the `trace()` function, where we loop over all the objects in the `objects` list.

```
bool trace( 
    const Vec3f &orig, const Vec3f &dir, 
    const std::vector<std::unique_ptr<Object>> &objects, 
    float &tNear, uint32_t &index, Vec2f &uv, Object **hitObject) 
{ 
    *hitObject = nullptr; 
    for (uint32_t k = 0; k < objects.size(); ++k) { 
        float tNearTriangle = kInfinity; 
        uint32_t indexTriangle; 
        Vec2f uvTriangle; 
        if (objects[k]->intersect(orig, dir, tNearTriangle, indexTriangle, uvTriangle) && tNearTriangle < tNear) { 
            *hitObject = objects[k].get(); 
            tNear = tNearTriangle; 
            index = indexTriangle; 
            uv = uvTriangle; 
        } 
    } 
 
    return (*hitObject != nullptr); 
} 
 
Vec3f castRay( 
    const Vec3f &orig, const Vec3f &dir, 
    const std::vector<std::unique_ptr<Object>> &objects, 
    const Options &options) 
{ 
    Vec3f hitColor = options.backgroundColor; 
    float tnear = kInfinity; 
    Vec2f uv; 
    uint32_t index = 0; 
    Object *hitObject = nullptr; 
    if (trace(orig, dir, objects, tnear, index, uv, &hitObject)) { 
        Vec3f hitPoint = orig + dir * tnear; 
        Vec3f hitNormal; 
        Vec2f hitTexCoordinates; 
        hitObject->getSurfaceProperties(hitPoint, dir, index, uv, hitNormal, hitTexCoordinates); 
        float NdotView = std::max(0.f, hitNormal.dotProduct(-dir)); 
        const int M = 10; 
        float checker = (fmod(hitTexCoordinates.x * M, 1.0) > 0.5) ^ (fmod(hitTexCoordinates.y * M, 1.0) < 0.5); 
        float c = 0.3 * (1 - checker) + 0.7 * checker; 
 
        hitColor = c * NdotView; 
    } 
 
    return hitColor; 
}
```

The `intersect()` method of the triangle mesh is finally called. To test if the ray intersects the mesh, we call the ray-triangle intersection test function for each triangle. As explained in the previous chapter, as we do so, we also keep track of the closest intersection distance in case the ray intersects more than one triangle (we also need to keep track of the intersect triangle index as well as the hit point barycentric coordinates. This information will be required for shading).

```
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
            isect = true; 
        } 
        j += 3; 
    } 
 
    return isect; 
}
```

Finally, if the ray intersects the mesh, we perform shading in the `castRay()` function (lines 47-56). We first call the `getGeometryData()` method of the intersected mesh to compute the normal and the texture coordinates at the intersection point. We then use this information to create the checkerboard pattern and calculate the facing ratio, which, once mixed, forms the final pixel color (line 56).

```
class TriangleMesh : public Object 
{ 
    ... 
    void getSurfaceProperties( 
        const Vec3f &hitPoint, 
        const Vec3f &viewDirection, 
        const uint32_t &triIndex, 
        const Vec2f &uv, 
        Vec3f &hitNormal, 
        Vec2f &hitTextureCoordinates) const 
    { 
        // face normal
        const Vec3f &v0 = P[trisIndex[triIndex * 3]]; 
        const Vec3f &v1 = P[trisIndex[triIndex * 3 + 1]]; 
        const Vec3f &v2 = P[trisIndex[triIndex * 3 + 2]]; 
        hitNormal = (v1 - v0).crossProduct(v2 - v0); 
        hitNormal.normalize(); 
 
        // texture coordinates
        const Vec2f &st0 = texCoordinates[triIndex * 3]; 
        const Vec2f &st1 = texCoordinates[triIndex * 3 + 1]; 
        const Vec2f &st2 = texCoordinates[triIndex * 3 + 2]; 
        hitTextureCoordinates = (1 - uv.x - uv.y) * st0 + uv.x * st1 + uv.y * st2; 
 
        // vertex normal
        /* 
        const Vec3f &n0 = N[triIndex * 3]; 
        const Vec3f &n1 = N[triIndex * 3 + 1]; 
        const Vec3f &n2 = N[triIndex * 3 + 2]; 
        hitNormal = (1 - uv.x - uv.y) * n0 + uv.x * n1 + uv.y * n2; 
        */ 
    } 
    ... 
}; 
 
Vec3f castRay( 
    const Vec3f &orig, const Vec3f &dir, 
    const std::vector<std::unique_ptr<Object>> &objects, 
    const Options &options) 
{ 
    Vec3f hitColor = options.backgroundColor; 
    float tnear = kInfinity; 
    Vec2f uv; 
    uint32_t index = 0; 
    Object *hitObject = nullptr; 
    if (trace(orig, dir, objects, tnear, index, uv, &hitObject)) { 
        Vec3f hitPoint = orig + dir * tnear; 
        Vec3f hitNormal; 
        Vec2f hitTexCoordinates; 
        hitObject->getSurfaceProperties(hitPoint, dir, index, uv, hitNormal, hitTexCoordinates); 
        float NdotView = std::max(0.f, hitNormal.dotProduct(-dir)); 
        const int M = 10; 
        float checker = (fmod(hitTexCoordinates.x * M, 1.0) > 0.5) ^ (fmod(hitTexCoordinates.y * M, 1.0) < 0.5); 
        float c = 0.3 * (1 - checker) + 0.7 * checker; 
 
        hitColor = c * NdotView; //Vec3f(uv.x, uv.y, 0); 
    } 
 
    return hitColor; 
}
```

A small note about shading: don't worry too much if you need help understanding the code related to shading. It will be explained in the introduction to shading. Note that the code is similar to the one we used in the lesson on rasterization. First, we use the barycentric coordinates of the hit point to interpolate the actual object st or texture coordinate. The final texture coordinates of the hit point are then used to compute a checkerboard pattern. As for normals, we have two options. Face normal, which we talked about in the rasterization lesson, can be calculated by taking the cross product of two of the triangle edges. We can also use the hit point barycentric coordinates to interpolate the vertex normals (read from the geometry file), the same way we interpolated texture coordinates. As mentioned, this code will be soon explained (check the lesson introduction to shading).

## Result

First, let's look at the visual result. As you can see, the image produced by the ray tracer is perfectly identical to the image produced by the rasterizer (besides the background color). As mentioned at the beginning of this lesson, mathematics never lies. If you do the right thing, the two rendering techniques should produce precisely the same result. We have at least proven that this was the case, which is also a way of validating that our programs do the right thing. Hooray!

![](/images/ray-triangle-mesh/cow-comparaison.gif?)

Now let's compare rendering time. Ray-tracing: 15.22 seconds. Rasterizer: 0.00628 seconds (we ran the two programs on the same machine). As expected, with a naive implementation, ray-tracing is 2421 times slower. This is a significant difference. Hopefully, as mentioned in the previous chapter, this can be improved using acceleration structures. We won't study acceleration structures in this section, but you can find lessons on this topic in the Advanced Ray-Tracing section. Despite being slow, we will keep using ray tracing in the following lessons to learn about shading. Ray-tracing, as explained before, is a much better technique for shading. Shading has to do with computing visibility between points in space, and ray tracing is well suited for that. It offers a unified framework to solve both the visibility problem and shading.

## Conclusion

Congratulation. At this point in your learning journey, you now know the difference between ray tracing and rasterization, how to implement the two algorithms, and produce an image of a 3D model viewed from a given viewpoint that is consistent regardless of the technique you use to render this image. You have also been able to judge the difference in performance between the two methods. In the next lesson, we will learn about transforming objects. We will then be ready to start our first lesson on shading.

## Exercises

Compute the object's bounding box and implement the ray-box intersection test to accelerate the render.