## Basic Maths Tools

Before we learn about techniques to solve the ray-triangle intersection test, let's first look at some of the tools we will use to solve this problem. As we said earlier, the triangle's vertices define a plane. Every plane has a normal, which can be seen as a line perpendicular to the plane's surface. Considering that we know the coordinates of the triangle's vertices (A, B, and C), we can compute the vectors AB and AC (which are simply the lines going from A to B and A to C). To do so, we subtract B from A and C from A.

To compute the plane's normal (the same as the triangle's normal since the triangle lies in the plane), we calculate the cross-product of AB and AC. Remember, these two vectors lie in the same plane since they connect the triangle's vertices. The vector resulting from this cross product (denoted N) is the normal we are looking for (Figure 1). These operations are summarized in this pseudocode example:

```
Vec3f v0(-1, -1, 0), v1(1, -1, 0), v2(0, 1, 0);
Vec3f A = v1 - v0; // edge 0
Vec3f B = v2 - v0; // edge 1
Vec3f C = cross(A, B); // this is the triangle's normal
normalize(C);
```

$$
\begin{array}{l}
A = v1 - v0 = (1--1, -1--1, 0) = (2, 0, 0)\\
B = v2 - v0 = (0--1, 1--1, 0) = (1, 2, 0)\\
C = A \times B = \\
C_x = A_y*B_z - A_z*B_y = 0\\
C_y = A_z*B_x - A_x*B_z = 0\\
C_z = A_x*B_y - A_y*B_x = 2*2 - 0*1 = 4\\
\end{array}
$$

If we normalize C, we get the vector (0, 0, 1) which, as you can see, is parallel to the positive z-axis, which is what we expect since the vertices v0, v1, and v2 lie in the XY-plane.

## Coordinate System Handedness

The order in which a triangle's vertices are defined affects the orientation of the surface's normal. V0, V1, and V2, created in counter-clockwise order (CCW), produce a normal denoted N. If the vertices were created in clockwise order (CW), the normal we would get from the cross product of the two edges (A=V1-V0 and B=V2-V0) would then point in the opposite direction to N. We would get -N.

![Figure 1: vector A and B can be computed from V1 - V0 and V2 - V0, respectively. The normal of the triangle, or vector C, is the cross product of A and B. Note that the order in which the vertices were created determines the direction of the normal (in this example, vertices have been created counter-clockwise).](/images/ray-triangle/triangle2.png?)

When you create a triangle in Maya (which uses a right-hand coordinate system), if you turn CCW as you generate the triangle's vertices, then the x-axis (V0V1) becomes A, the y-axis (V0V2) becomes B and C, the cross product AxB, is parallel to the z-axis (Figure 2). But if create vertices in clockwise order, then C will point along the negative z-axis (creating vertices in CW order and computing N from the cross product AxB is the same as computing the cross product BxA when the vertices are created in CCW with A=V1-V0 and B=V2-V0).

![Figure 2: creating the vertices in CCW or CW changes the direction of the normal.](/images/ray-triangle/trirh.png?)

Now imagine that you create this triangle in a left-hand coordinate system from three vertices that have the same coordinates as the triangle from the right-hand coordinate system example (V0 = (0,0,0), V1=(1,0,0) and V2=(0,1,0)). Following the same steps, we can create vectors A (V1-V0) and B (V2-V0) and compute C (from AxB). The result is equal to (0,0,1) as with the right-hand coordinate system example (the vertices coordinates are the same. Therefore, vectors A and B are the same as the result of the cross product AxB). Remember that by convention, the z-axis in the left-hand coordinate system is pointing in the direction of the screen (when the x-axis is pointing to the right). Again, everything is coherent with the explanations we have given so far about the handedness of coordinate systems. The cross-product result, as we mentioned in the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/math-operations-on-points-and-vectors), is an anti-commutative operation.

![Figure 3: triangles created in left- or right-hand coordinate system don't look the same when rendered from the same camera (pointing down the negative z-axiz).](/images/ray-triangle/trilhrh1.png?)

Now the next step is to render each of these triangles from a camera pointing down the negative z-axis. The result of these tests is shown in Figure 4. The two resulting images don't look the same even though the triangle's vertices have the same coordinates, which is not what we want. The choice of a coordinate system's handedness should not change how geometry is seen from the camera's point of view. A person rendering an image probably wants to see the same image of the same triangle independently from the handedness being used. Even though this is not directly related to the topic of the ray-triangle intersection test, this is a significant problem you need to know about. Usually, we solve it by mirroring the camera along the negative z-axis (both its position and orientation), as shown in Figure 4.

![Figure 4: to get the same image of the triangle, we need to mirror the camera about the z-axis and flip the triangle's normal.](/images/ray-triangle/mirrorcamera.png?)

This is working. However, you can see that now the direction of the camera and the direction of the normal have changed compared to what we had before mirroring the camera. In Figure 3, the camera's direction and the triangle's normal point are opposite. But in Figure 4, they are now pointing in the same direction. These two vectors (the triangle's normal and the camera direction) play an essential role in shading; therefore, we do not change their direction about each other. Thus, the solution to the flipped triangle problem is to mirror the camera and flip the normal direction. Remember, though, that this is only necessary if you modeled the triangle in a modeling application using a left-hand coordinate system but render it in a renderer using a right-hand coordinate system (or vice-versa, which is the case if you generate geometry in Maya and render this geometry in RenderMan which is using a right- and left-hand coordinate system respectively).

<details>
The combination of order and direction in which the vertices are specified is called winding.
</details>

We now have all the tools to solve the ray-triangle intersection test using simple geometry.