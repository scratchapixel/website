## Single vs. Double-Sided Triangle

![Figure 1: The outside surface of a primitive is the side from which the normal points outward; the inside surface is the opposite side.](/images/ray-triangle/frontback.png?)

We already mentioned the term back-face culling in the lesson on rasterization but let's introduce it one more time. Let's say that to develop a routine that computes the intersection of a ray with a triangle, we create a scene using a right-hand coordinate system where the triangle's vertices are declared in counter-clockwise order. The triangle is five units away from the ray's origin, located at the center of the world coordinate system (0, 0, 0). The ray is oriented along the negative z-axis. We know how to compute the triangle's normal, which by construction is oriented along the z-axis if all the triangles vertices lie in the x-y plane or a plane parallel to it (remember that the orientation of the normal depends on the coordinate system's handedness and the triangle's vertices creation order, also called winding). As you see, the ray direction and the normal triangle face each other (Figure 1).

The surface of a primitive whose normal is pointing outward is defined as the outside surface. The opposite side is defined as the inside surface. If you change the coordinate system handedness or the vertices creation order, you will flip the normal direction. Consequently, the inside surface of a primitive will become the outside surface, and the outside surface the inside surface. The surface orientation has implications for surface visibility and shading. We are not concerned about shading yet, but let's look at the visibility problem.

In a CG scene, you may have the case of primitives for which the outside surface is facing the camera. The surface is said to be **front-facing**. In the opposite case (when the outside surface of a primitive is facing away from the camera), the surface is said to be **back-facing**. Rendering programs often allow you to discard, during the visibility part of the rendering process, any primitive whose outside surface is not facing the camera (back-facing), making only visible primitives whose outside surface is facing the camera (front-facing). OpenGL, for example, has such controls. It is said that such primitives are **single-sided** (single-sided primitives are only visible when their outside surface faces the viewer). If you wish to make a primitive visible in both cases (when the object's surface is facing the camera or in the opposite direction), you may have to declare the primitive as **double-sided**. Some rendering programs do not give the option to the user to define primitives as being either single or double-sided. They will either systematically discard all the back-facing primitives (they won't be visible in the final frame) or make them all double-sided by default. In the RenderMan specifications, you can declare on a primitive basis if an object should be treated as a single or double-sided primitive (using the RiSide call). Note that the RenderMan specifications also give you the option to specify the coordinate system handedness of the scene (with the `RiOrientation` procedure) and reverse the orientation of a primitive's surface in regards to the coordinate system handedness (if desired) with the `RiReverseOrientation` call.

<details>

![](/images/ray-triangle/culling.png?)

The term **back-face bulling** (synonymous with removing if you prefer) means that objects whose normals point away from the view direction will not be drawn to the screen. Turning back-face culling on means that polygons, triangles, surfaces, etc., whose normal are not facing the view direction, won't be rendered. Only geometry whose normals face the camera will be visible in the final image. This technique can lead to significant speedup (and memory savings) in z-buffer style renderers because it can reduce the number of surfaces being rendered by a substantial factor (for example, for a polygon sphere, in theory, 50% of the faces could be culled). With ray tracing, this feature is not as helpful. Generally, with ray tracing, we want geometry in the scene to cast shadows, for example, regardless of the orientation of the object's surface with resepct to the ray direction; however, the backface culling option is still desired for primary rays. If the polygon's surface normal points more than 90 degrees from the viewing vector (see adjacent figure), we could avoid testing this polygon for intersections. A simple dot product test between the normal and the view direction is enough to determine whether the face should be culled. Of course, we don't cull the face if the object it belongs to is declared **double-sided**, and culling back face surfaces when objects are transparent is also not necessarily desirable.

The image above shows the faces (in orange) that will be culled at render time if the object is not transparent, rendered as a single-sided object, and if back-face culling is turned on.
</details>

If you develop a rendering program, you should be very careful about making sure it handles all possible cases. It is best to give users the ability to choose the coordinate system handedness, reverse the surface orientation if needed, and define if primitives are single or double-sided. For the same geometry, these settings can change what's visible in the frame. For example, the choice of coordinate system handedness changes the direction on the normal defined by the vertices V0, V1, V2\. When you use a right-hand coordinate system, the normal (computed from the triangle's vertices) points away from the camera; the ray-triangle intersection routine will return false if back-face culling is turned on, even if the ray intersects the triangle. Switching to a left-hand coordinate system would change the normal's orientation, and the test, in this case, would be successful. In the first case, the triangle wouldn't be visible, but in the second, it would, even though the triangle is the same.

Here is how you would implement the single/double-sided feature. At the beginning of the function, we compute the dot product of the triangle's normal with the ray direction. If this dot product is lower than 0, the two vectors point in opposite directions. Thus, the surface is front-facing. If the dot product is greater than 0, the vectors point in the same direction. We are looking at the inside of the surface (or the back of it, it's back-facing). If the primitive was declared single-sided, it shouldn't then be visible. In this case, the function returns false.

```
bool intersectTriangle(point v0, ..., const bool &isSingleSided)
{
    Vec3f e0 = v1 - v0;
    Vec3f e1 = v2 - v0;
    Vec3f N = crossProduct(e0, e1);
    normalize(N);
    ...
    // implementing the single/double-sided feature
    if (dotProduct(dir, N) &gt; 0 && isSingleSided)
        return false; // back-facing surface
    ...
    return true;
}
```