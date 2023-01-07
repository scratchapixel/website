## Ray-triangle intersection: geometric solution

![Figure 1: the intersection of a ray and a triangle. The triangle lies in a plane. The value \(t\) is the distance from the ray origin to the intersection point.](/images/ray-triangle/triray2.png?)

In the previous paragraphs, we learned how to calculate a plane's normal. Next, we need to find out the position of point P (for some illustrations, we also used Phit), the point where the ray intersects the plane.

### Step 1: Finding P

We know that P is somewhere on the ray defined by its origin \(O\) and its direction \(R\). We used \(D\) in the previous lesson, but we will use the term \(R\) in this lesson to avoid confusion with the term \(D\) from the plane equation. The ray parametric equation is (equation 1):

$$P=O + tR.$$

Where \(t\) is the distance from the ray origin \(O\) to P. To find P, we must find \(t\) (Figure 1). What else do we know? We know the plane's normal, which we have already computed, and the plane equation (2), which is (check the chapter on the ray-plane intersection from the [previous lesson](/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-plane-and-ray-disk-intersection) for more information on this topic):

$$
\begin{array}{l}
Ax + By + Cz + D = 0\\
D = -(Ax + By + Cz)
\end{array}
$$

Where A, B, C can be seen as the components (or coordinates) of the normal to the plane (\N_{plane}=(A, B, C)\), and \(D\) is the distance from the origin (0, 0, 0) to the plane (if we trace a line from the origin to the plane, parallel to the plane's normal). The variables x, y, and z are the coordinates of any point on this plane.

We know the plane's normal and that the three triangle's vertices (V0, V1, V2) lie in the plane. It is, therefore, possible to compute \(D\). Any of the three vertices can be chosen. Let's choose V0:

```
float D = -dotProduct(N, v0);
// or if you want to compute the dot product directly
float D = -(N.x * v0.x + N.y * v0.y + N.z * v0.z);
```

We also know that point P is the intersection point of the ray, and the plane lies in the plane. Consequently, we can substitute @@\b(x, y, z)@@ (equation 2) for  @@\rP@@ or @@\gO + tR@@ that P is equal to (equation 1) and solve for t (equation 3):

$$
\begin{array}{l}
\textcolor{red}{P} = \textcolor{green}{O + tR}\\
A\textcolor{blue}{x} + B\textcolor{blue}{y} + C\textcolor{blue}{z} + D = 0\\
A * \textcolor{red}{P_x} + B * \textcolor{red}{P_y} + C * \textcolor{red}{P_z} + D = 0\\
A * \textcolor{green}{(O_x + tR_x)} + B * \textcolor{green}{(O_y + tR_y)} + C * \textcolor{green}{(O_z + tR_z)} + D = 0\\
A * O_x + B * O_y + C * O_z + A * tR_x + B * tR_y + C * tR_z + D = 0\\
t * (A * R_x + B * R_y + C * R_z) + A * O_x + B * O_y + C * O_z + D = 0\\
t = -{\dfrac{A * O_x + B * O_y + C * O_z + D}{A * R_x + B * R_y + C * R_z}}\\
t = -{\dfrac{ N(A,B,C) \cdot O + D}{N(A,B,C) \cdot R}}
\end{array}
$$

```
float t = - (dot(N, orig) + D) / dot(N, dir);
```

We now have computed \(t\), which we can use to calculate the position of P:

```
Vec3f Phit = orig + t * dir;
```

There are two very important cases that we need to look at before we check if the point is inside the triangle.

### The Ray And The Triangle Are Parallel

If the ray and the plane are parallel, they won't intersect (Figure 2). For robustness, we need to handle that case if it happens. This is very simple. If the triangle and the ray are parallel, then the triangle's normal and the ray's direction should be perpendicular.

![Figure 2: several situations can occur. The ray can intersect the triangle or miss it. If the ray is parallel to the triangle, there is no possible intersection. This situation occurs when the normal of the triangle and the ray direction is perpendicular (and the dot product of these two vectors is 0).](/images/ray-triangle/trirays.png?)

We have learned that the dot product of two perpendicular vectors is 0. If you look at the denominator of equation 3 (the term below the line), we already compute the dot product between the triangle's normal N and the ray direction \(D\). Our code is not very robust because this term can potentially be 0, and we should always catch a possible division by 0. When this term equals 0, the ray is parallel to the triangle. Since they don't intersect in that case, there's no need to compute \(t\) anyway. Conclusion: before we calculate \(t\), we will calculate the result of the term \(N \cdot R\) first, and if the result is 0, the function will return false (no intersection).

### The Triangle is "Behind" the Ray

![Figure 3: If a triangle is "behind" the ray, it shouldn't be visible. Whenever the value of t computed with equation 3 is lower than 0, the intersection point is located behind the ray's origin and should be discarded. There is no intersection in that case.](/images/ray-triangle/tribehind.png?)

So far, we have assumed that the triangle was always in front of the ray. But what happens if the triangle is behind the ray with the ray still looking in the same direction? Usually, the triangle shouldn't be visible. Equation 3 returns a valid result, even when the triangle is "behind" the ray. In that case, \(t\) is negative, which causes the intersection point to be in the opposite direction to the ray direction. If we don't catch this "error," the triangle will show up in the final image, which we don't want. Therefore we need to check the sign of \(t\) before deciding whether the intersection is valid. If \(t\) is lower than 0, the triangle is behind the ray's origin (with regards to the ray's direction) and is not visible. There is no intersection, and we can return false again. If \(t\) is greater than 0, the triangle is "visible" to that ray, and we can proceed to the next step.

### Step 2: Is P Inside or Outside the Triangle?

Now that we have found the point P, which is the point where the ray and the plane intersect, we still have to find out if P is inside the triangle (in which case the ray intersects the triangle) or if P is outside (in which case the rays misses the triangle). Figure 2 illustrates these two cases.

![Figure 4: C and C' point in opposite directions.](/images/ray-triangle/triinsideout1.png?)

![Figure 5: if P is on the left side of A, the dot product N.C is positive. If P is on the right side (P'), N.C' is negative. The vector C is computed from v0 and P (C=P-v0)](/images/ray-triangle/triinsideout2.png?)

![Figure 6: to find out if P is inside the triangle, we can test if the dot product of the vector along the edge and the vector defined by the first vertex of the tested edge and P is positive (meaning P is on the left side of the edge). If P is on the left of all three edges, then P is inside the triangle.](/images/ray-triangle/triinsideout3.png?)

The solution to this problem is simple and called the **inside-outside** test (we have already used this term in the lesson on rasterization. In the context of rasterization, the test was used for 2D triangles. We will use it here for 3D triangles. Imagine having a vector A aligned with the x-axis (Figure 4). Let's imagine that this vector is aligned with one edge of our triangle (the edge defined by the two vertices v0-v01). B, the second edge, is determined by the vertices v0 and v2 of the triangles, as shown in Figure 4. Let's calculate the cross-product of these two vectors. As expected, the result is a vector pointing in the same direction as the z-axis and as the normal of the triangle.

$$
\begin{array}{l}
A=(1, 0, 0)\\
B=(1, 1, 0)\\
C_x = A_y B_z - A_z B_y = 0\\
C_y = A_z B_x - A_x B_z = 0\\
C_z = A_x B_y - A_y B_x = 1 * 1 - 0 * 1 = 1\\
C = (0, 0, 1)
\end{array}
$$

Let's now imagine that rather than having the coordinates (1, 1, 0), the vertex v2 has the coordinates (1, -1, 0). In other words, we have mirrored its position across the x-axis. If we compute the cross product AxB', we get C'=(0, 0, -1).

$$
\begin{array}{l}
A=(1, 0, 0)\\
B=(1, -1, 0)\\
C_x = A_y B_z - A_z B_y = 0\\
C_y = A_z B_x - A_x B_z = 0\\
C_z = A_x B_y - A_y B_x = 1 * -1 - 0 * 1 = -1\\
C = (0, 0, -1)
\end{array}
$$

Because C and N point in the same direction, their dot product returns a value greater than 0 (positive). However, because C' and N are pointing in opposite directions, their dot product returns a value lower than 0 (negative). What does that test tell us? We know that the point where the ray intersects the triangle, and the triangle are in the same plane. We also know from the test we have just made that if a point P which is in the triangle's plane (such as the vertex V2 or the intersection point), is on the left side of vector A, then the dot product between the triangle's normal and vector C is positive (C is the result of the cross product between A and B. In this scenario, A = (v1 - v0) and B = (P - v0)). However, if P is on the right side of A (as with v2'), this dot product is negative. You can see in Figure 5 that point P is inside the triangle when it is located on the left side of A. To apply the technique we have just described to the ray-triangle intersection problem, we need to repeat the left/right test for each triangle edge. If for each triangle's edges, we find that point P is on the left side of vector C (where C is defined as v1-v0, v2-v1, and v0-v2, respectively, for each edge of the triangle), then we know for sure that P is inside the triangle. If the test fails for any triangle edges, P lies outside the triangle's boundaries. This process is illustrated in Figure 6.

Here is an example of the inside-outside test in pseudocode:

```
Vec3f edge0 = v1 - v0;
Vec3f edge1 = v2 - v1;
Vec3f edge2 = v0 - v2;
Vec3f C0 = P - v0;
Vec3f C1 = P - v1;
Vec3f C2 = P - v2;
if (dotProduct(N, crossProduct(edge0, C0)) &gt; 0 &amp;&amp; 
    dotProduct(N, crossProduct(edge1, C1)) &gt; 0 &amp;&amp;
    dotProduct(N, crossProduct(edge2, C2)) &gt; 0) return true; // P is inside the triangle
```

<details>
Note the similarities between this method and the method we used in the rasterization lesson to find if a pixel overlaps a (2D) triangle.
</details>

Let's write the complete ray-triangle intersection test routine code. First, we will compute the triangle's normal, then test if the ray and the triangle are parallel. If they are, the intersection test fails. If they are not parallel, we compute \(t\), from which we can compute the intersection point P. If the inside-out test succeeds (we test if P is on the left side of each triangle's edges), then the ray intersects the triangle, and P is inside the triangle's boundaries. The test is successful.

```
bool rayTriangleIntersect(
    const Vec3f &orig, const Vec3f &dir,
    const Vec3f &v0, const Vec3f &v1, const Vec3f &v2,
    float &t)
{
    // compute the plane's normal
    Vec3f v0v1 = v1 - v0;
    Vec3f v0v2 = v2 - v0;
    // no need to normalize
    Vec3f N = v0v1.crossProduct(v0v2); // N
    float area2 = N.length();
 
    // Step 1: finding P
    
    // check if the ray and plane are parallel.
    float NdotRayDirection = N.dotProduct(dir);
    if (fabs(NdotRayDirection) < kEpsilon) // almost 0
        return false; // they are parallel, so they don't intersect! 

    // compute d parameter using equation 2
    float d = -N.dotProduct(v0);
    
    // compute t (equation 3)
    t = -(N.dotProduct(orig) + d) / NdotRayDirection;
    
    // check if the triangle is behind the ray
    if (t < 0) return false; // the triangle is behind
 
    // compute the intersection point using equation 1
    Vec3f P = orig + t * dir;
 
    // Step 2: inside-outside test
    Vec3f C; // vector perpendicular to triangle's plane
 
    // edge 0
    Vec3f edge0 = v1 - v0; 
    Vec3f vp0 = P - v0;
    C = edge0.crossProduct(vp0);
    if (N.dotProduct(C) < 0) return false; // P is on the right side
 
    // edge 1
    Vec3f edge1 = v2 - v1; 
    Vec3f vp1 = P - v1;
    C = edge1.crossProduct(vp1);
    if (N.dotProduct(C) < 0)  return false; // P is on the right side
 
    // edge 2
    Vec3f edge2 = v0 - v2; 
    Vec3f vp2 = P - v2;
    C = edge2.crossProduct(vp2);
    if (N.dotProduct(C) < 0) return false; // P is on the right side;

    return true; // this ray hits the triangle
}
```

<details>

The "inside-outside" technique we have just described works for any [convex polygon](http://en.wikipedia.org/wiki/Convex_polygon). Repeat the method we have used for triangles for each edge of the polygon. Compute the cross product of the vector defined by the two edges' vertices and the vector defined by the first edge's vertex and the point. Compute the dot product of the resulting vector and the polygon's normal. The sign of the resulting dot product determines if the point is on the right or left side of that edge. Iterate through each edge of the polygon. There's no need to test the other edges if one fails to pass the test.
</details>

Note that this technique can be made faster if the triangle's normal and the value \(D\) from the plane equation are precomputed and stored in memory for each triangle of the scene.

## What's next?

In this chapter, we have presented a technique to compute the ray-triangle intersection test using simple geometry. However, there is much more to the ray-triangle intersection test, which we haven't considered yet, such as whether the ray hits the triangle from the front or the back. We can also compute what we call the intersection point's barycentric coordinates. These coordinates are necessary for doing things such as applying a texture to the triangle.