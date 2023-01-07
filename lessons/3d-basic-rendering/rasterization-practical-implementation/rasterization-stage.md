## Rasterization: What Are We Trying to Solve?

Rasterization is the process by which a primitive is converted to a two-dimensional image. Each point of this image contains such information as color and depth. Thus, rasterizing a primitive consists of two parts. The first is to determine which squares of an integer grid in window coordinates are occupied by the primitive. The second is assigning a color and a depth value to each such square. (OpenGL Specifications)

![Figure 1: by testing, if pixels in the image overlap the triangle, we can draw an image of that triangle. This is the principle of the rasterization algorithm.](/images/rasterization/rasterization-triangle1.png?)

In the previous chapter, we learned how to perform the first step of the rasterization algorithm in a way, which is to project the triangle from 3D space onto the canvas. This definition is not entirely accurate in fact, since what we did was to transform the triangle from camera space to screen space, which as mentioned in the previous chapter, is also a three-dimensional space. However the x- and y-coordinates of the vertices in screen-space correspond to the position of the triangle vertices on the canvas, and by converting them from screen-space to NDC space and then finally from NDC-space to raster-space, what we get in the end are the vertices 2D coordinates in raster space. Finally, we also know that the z-coordinates of the vertices in screen-space hold the original z-coordinate of the vertices in camera space (inverted so that we deal with positive numbers rather than negative ones).

What we need to do next, is to loop over the pixel in the image and find out if any of these pixels overlap the "projected image of the triangle" (figure 1). In graphics APIs specifications, this test is sometimes called the **inside-outside test** or the **coverage test**. If they do, we then set the pixel in the image to the triangle's color. The idea is simple but of course, we now need to come up with a method to find if a given pixel overlaps a triangle. This is essentially what we will study in this chapter. We will learn about the method that is typically used in rasterization to solve this problem. It uses a technique known as the **edge function** which we are now going to describe and study. This edge function is also going to provide valuable information about the position of the pixel within the projected image of the triangle known as **barycentric coordinates**. Barycentric coordinates play an essential role in computing the actual depth (or the z-coordinate) of the point on the surface of the triangle that the pixel overlaps. We will also explain what barycentric coordinates are in this chapter and how they are computed.

At the end of this chapter, you will be able to produce a very basic rasterizer. In the next chapter, we will look into the possible issues with this very naive implementation of the rasterization algorithm. We will list what these issues are as well as study how they are typically addressed.

A lot of research has been done to optimize the algorithm. The goal of this lesson is not to teach you how to write or develop an optimized and efficient renderer based on the rasterization algorithm. The goal of this lesson is to teach the basic principles of the rendering technique. Don't think though that the techniques we present in these chapters are not used. They are used to some extent, but how they are implemented either on the GPU or in a CPU version of a production renderer, is just likely to be a highly optimized version of the same idea. What is truly important is to understand the principle and how it works in general. From there, you can study on your own the different techniques which are used to speed up the algorithm. But the techniques presented in this lesson are generic and make up the foundations of any rasterizer.

Keep in mind that drawing a triangle (since the triangle is a primitive we will use it in this case), is a two steps problem:

- We first need to find which pixels overlap the triangle.
- We then need to define which colors should the pixels overlapping the triangle be set to, a process that is called **shading**

The rasterization stage deals essentially with the first step. The reason we say essentially rather than exclusively is that at the rasterization stage, we will also compute something called **barycentric coordinates** which to some extent, are used in the second step.

## The Edge Function

As mentioned above, they are several possible methods to find if a pixel overlaps a triangle. It would be good to document older techniques, but in this lesson, will only present the method that is generally used today. This method was presented by Juan Pineda in 1988 and a paper called "**A Parallel Algorithm for Polygon Rasterization**" (see references in the last chapter).

![Figure 2: the principle of Pineda's method is to find a function, so that when we test on which side of this line a given point is, the function returns a positive number when it is to the left of the line, a negative number when it is to the right of this line, and zero when the point is exactly on the line.](/images/rasterization/rasterization-triangle2.png?)

![Figure 3: points contained within the white area are all located to the right of all three edges of the triangle.](/images/rasterization/rasterization-triangle3.png?)

Before we look into Pineda's technique itself, we will first describe the principle of his method. Let's say that the edge of a triangle can be seen as a line splitting the 2D plane (the plane of the image) in two (as shown in figure 2). The principle of Pineda's method is to find a function which he called the **edge function**, so that when we test on which side of this line a given point is (the point P in figure 2), the function returns a negative number when it is to the left of the line, a positive number when it is to the right of this line, and zero when the point is exactly on the line.

In figure 2, we applied this method to the first edge of the triangle (defined by the vertices v0-v1. Be careful the order is important). If we now apply the same method to the two other edges (v1-v2 and v2-v0), we then can see that there is an area (the white triangle) within which all points are positive (figure 3). If we take a point within this area, then we will find that this point is to the right of all three edges of the triangle. If P is a point in the center of a pixel, we can then use this method to find if the pixel overlaps the triangle. If for this point, we find that the edge function returns a positive number for all three edges, then the pixel is contained in the triangle (or may lie on one of its edges). The function Pinada uses also happens to be linear which means that it can be computed incrementally but we will come back to this point later.

Now that we understand the principle, let's find out what that function is. The edge function is defined as (for the edge defined by vertices V0 and V1):

$$E_{01}(P) = (P.x - v0.x) * (V1.y - V0.y) - (P.y - V0.y) * (V1.x - V0.x).$$

As the paper mentions, this function has the useful property that its value is related to the position of the point (x,y) relative to the edge defined by the points V0 and V1:

- E(P) > 0 if P is to the "right" side
- E(P) = 0 if P is exactly on the line
- E(P) < 0 if P is to the "left " side

This function is equivalent in mathematics to the magnitude of the cross products between the vector (v1-v0) and (P-v0). We can also write these vectors in a matrix form (presenting this as a matrix has no other interest than just neatly presenting the two vectors):

$$
\begin{vmatrix}
(P.x - V0.x) & (P.y - V0.y) \\
(V1.x - V0.x) & (V1.y - V0.y)
\end{vmatrix}
$$

If we write that \(A = (P-V0)\) and \(B = (V1 - V0)\), then we can also write the vectors A and B as a 2x2 matrix:

$$
\begin{vmatrix}
 A.x & A.y \\ 
 B.x & B.y
\end{vmatrix}
$$

The [**determinant**](http://en.wikipedia.org/wiki/Determinant#2.C2.A0.C3.97.C2.A02_matrices) of this matrix can be computed as:

$$A.x * B.y - A.y * B.x.$$

If you now replace the vectors A and B with the vectors (P-V0) and (V1-V0) back again, you get:

$$(P.x - V0.x) * (V1.y - V0.y) - (P.y - V0.y) * (V1.x - V0.x).$$

Which as you can see, is similar to the edge function we have defined above. In other words, the edge function can either be seen as the determinant of the 2x2 matrix defined by the components of the 2D vectors (P-v0) and (v1-v0) or also as the magnitude of the cross product of the vectors (P-V0) and (V1-V0). Both the determinant and the magnitude of the cross-product of two vectors have the same geometric interpretation. Let's explain.

![Figure 4: the cross-product of vector B (blue) and A (red) gives a vector C (green) perpendicular to the plane defined by A and B (assuming the right-hand rule convention). The magnitude of vector C depends on the angle between A and B. It can either be positive or negative.](/images/rasterization/cross-product-anim.gif?)

![Figure 5: the area of the parallelogram is the absolute value of the determinant of the matrix formed by the vectors A and B (or the magnitude of the cross-product of the two vectors B and A (assuming the right-hand rule convention).](/images/rasterization/cross-product.png?)

![Figure 6: the area of the parallelogram is the absolute value of the determinant of the matrix formed by the vectors A and B. If the angle \(\theta\) is lower than \(\pi\) then the "signed" area is positive. If the angle is greater than \(\pi\) then the "signed" area is negative. The angle is computed with respect to the Cartesian coordinates defined by the vectors A and D. They can be seen to separate the plane in two halves.](/images/rasterization/cross-product1.png?)

![Figure 7: P is contained in the triangle if the edge function returns a positive number for the three indicated pairs of vectors.](/images/rasterization/cross-product2.gif?)

Understanding what's happening is easier when we look at the result of a cross-product between two 3D vectors (Figure 4). In 3D, the cross-product returns another 3D vector that is perpendicular (or orthonormal) to the two original vectors. But as you can see in Figure 4, the magnitude of that orthonormal vector also changes with the orientation of the two vectors with respect to each other. In Figure 4, we assume a right-hand coordinate system. When the two vectors A (red) and B (blue) are either pointing exactly in the same direction or opposite directions, the magnitude of the third vector C (in green) is zero. Vector A has coordinates (1,0,0) and is fixed. When vector B has coordinates (0,0,-1), then the green vector, vector C has coordinates (0,-1,0). If we were to find its "signed" magnitude, we would find that it is equal to -1. On the other hand, when vector B has coordinates (0,0,1), then C has coordinates (0,1,0) and its signed magnitude is equal to 1. In one case the "signed" magnitude is negative, and in the second case, the signed magnitude is positive. In fact, in 3D, the magnitude of a vector can be interpreted as the area of the parallelogram having A and B as sides as shown in Figure 5 (read the Wikipedia article on the [cross product](http://en.wikipedia.org/wiki/Cross_product#Geometric_meaning) to get more details on this interpretation):

$$Area = || A \times B || = ||A|| ||B|| \sin(\theta).$$

An area should always be positive, though the sign of the above equation provides an indication of the orientation of the vectors A and B with respect to each other. When with respect to A, B is within the half-plane defined by vector A and a vector orthogonal to A (let's call this vector D; note that A and D form a 2D Cartesian coordinate system), then the result of the equation is positive. When B is within the opposite half plane, the result of the equation is negative (Figure 6). Another way of explaining this result is that the result is positive when the angle \(\theta\) is in the range \(]0,\pi[\) and negative when \(\theta\) is in the range \(]\pi, 2\pi[\). Note that when \(\theta\) is exactly equal to 0 or \(\pi\) then the cross-product or the edge function returns 0.

To find if a point is inside a triangle, all we care about really is the sign of the function we used to compute the area of the parallelogram. However, the area itself also plays an important role in the rasterization algorithm; it is used to compute the barycentric coordinates of the point in the triangle, a technique we will study next. The cross-product in 3D and 2D has the same geometric interpretation, thus the cross-product between two 2D vectors also returns the "signed" area of the parallelogram defined by the two vectors. The only difference is that in 3D, to compute the area of the parallelogram you need to use this equation:

$$Area = || A \\times B || = ||A|| ||B|| \\sin(\\theta),$$

while in 2D, this area is given by the cross-product itself (which as mentioned before can also be interpreted as the determinant of a 2x2 matrix): 

$$Area = A.x * B.y - A.y * B.x.$$

From a practical point of view, all we need to do now is test the sign of the edge function computed for each edge of the triangle and another vector defined by a point and the first vertex of the edge (Figure 7).

$$
\begin{array}{l}
E_{01}(P) = (P.x - V0.x) * (V1.y - V0.y) - (P.y - V0.y) * (V1.x - V0.x),\\
E_{12}(P) = (P.x - V1.x) * (V2.y - V1.y) - (P.y - V1.y) * (V2.x - V1.x),\\
E_{20}(P) = (P.x - V2.x) * (V0.y - V2.y) - (P.y - V2.y) * (V0.x - V2.x).
\end{array}
$$

If all three tests are positive or equal to 0, then the point is inside the triangle (or lying on one of the edges of the triangle). If any one of the tests is negative, then the point is outside the triangle. In code we get:

```
bool edgeFunction(const Vec2f &a, const Vec3f &b, const Vec2f &c)
{
    return ((c.x - a.x) * (b.y - a.y) - (c.y - a.y) * (b.x - a.x) &gt= 0);
}

bool inside = true;
inside &= edgeFunction(V0, V1, p);
inside &= edgeFunction(V1, V2, p);
inside &= edgeFunction(V2, V0, p);

if (inside == true) {
    // point p is inside triangles defined by vertices v0, v1, v2
    ...
}
```

<details>
The edge function has the property of being linear. We refer you to the original paper if you wish to learn more about this property and how it can be used to optimize the algorithm. In short though, let's say that because of this property, the edge function can be run in parallel (several pixels can be tested at once). This makes the method ideal for hardware implementation. This explains partially why pixels on the GPU are generally rendered as a block of 2x2 pixels (pixels can be tested in a single cycle). Hint: you can also use SSE instructions and multi-threading to optimize the algorithm on the CPU.
</details>

## Alternative to the Edge Function

There are other ways than the edge function method to find if pixels overlap triangles, however as mentioned in the introduction of this chapter, we won't study them in this lesson. Just for reference though, the other common technique is called scanline rasterization. It is based on the Brenseham algorithm that is generally used to draw lines. GPUs use the edge method mostly because it is more generic than the scanline approach which is also more difficult to run in parallel than the edge method, but we won't provide more information on this topic in this lesson.

## Be Careful! Winding Order Matters

![Figure 8: clockwise and counter-clockwise winding.](/images/rasterization/winding.png?)

One of the things we have been talking about yet, but which has great importance in CG, is the order in which you declare the vertices making up the triangles. They are two possible conventions which you can see illustrated in Figure 8: **clockwise** or **counter-clockwise ordering** or **winding**. Winding is important because it essentially defines one important property of the triangle which is the orientation of its normal. Remember that the normal of the triangle can be computed from the cross product of the two vectors A=(V2-V0) and B=(V1-V0). Let's say that V0={0,0,0}, V1={1,0,0} and V2={0,-1,0} then (V1-V0)={1,0,0} and (V2-V0)={0,-1,0}. Let's now compute the cross-product of these two vectors:

$$
\begin{array}{l}
N = (V1-V0) \times (V2-V0)\\
N.x = a.y*b.z - a.z * b.y = 0*0 - 0*-1\\
N.y = a.z*b.x - a.x * b.z = 0*0 - 1*0\\
N.z = a.x*b.y - a.y * b.x = 1*-1 - 0*0 = -1\\
N=\{0,0,-1\}
\end{array}
$$

However if you declare the vertices in counter-clockwise order, then V0={0,0,0}, V1={0,-1,0} and V2={1,0,0}, (V1-V0)={0,-1,0} and (V2-V0)={1,0,0}. Let's compute the cross-product of these two vectors again:

$$
\begin{array}{l}
N = (V1-V0) \times (V2-V0)\\
N.x = a.y*b.z - a.z * b.y = 0*0 - 0*-1\\
N.y = a.z*b.x - a.x * b.z = 0*0 - 1*0\\
N.z = a.x*b.y - a.y * b.x = 0*0 - -1*1 = 1\\
N=\{0,0,1\}
\end{array}
$$

![Figure 9: the ordering defines the orientation of the normal.](/images/rasterization/winding1.png?)

![Figure 10: the ordering defines if points inside the triangle are positive or negative.](/images/rasterization/rasterization-triangle4.png?)

As expected, the two normals are pointing in opposite directions. The orientation of the normal has great importance for lots of different reasons, but one of the most important ones is called **face culling**. Most rasterizers and even ray-tracer for that matter may not render triangles whose normal is facing away from the camera. This is called [back-face culling](http://en.wikipedia.org/wiki/Back-face_culling). Most rendering APIs such as OpenGL or DirectX give the option to turn back-face culling off, however, you should still be aware that vertex ordering plays a role in what's rendered, among many other things. And not surprisingly, the edge function is one of these other things. Before we get to explain why it matters in our particular case, let's say that there is no particular rule when it comes to choosing the order. In reality, so many details in a renderer implementation may change the orientation of the normal that you can't assume that by declaring vertices in a certain order, you will get the guarantee that the normal will be oriented a certain way. For instance, rather than using the vectors (V1-V0) and (V2-V0) in the cross-product, you could as have used (V0-V1) and (V2-V1) instead. It would have produced the same normal but flipped. Even if you use the vectors (V1-V0) and (V2-V0), [remember](/lessons/mathematics-physics-for-computer-graphics/geometry/math-operations-on-points-and-vectors) that the order of the vectors in the cross-product changes the sign of the normal: \(A \times B=-B \times A\). So the direction of your normal also depends on the order of the vectors in the cross-product. For all these reasons, don't try to assume that declaring vertices in one order rather than the other will give you one result or the other. What's important though, is that once you stick to the convention you have chosen. Generally, graphics APIs such as OpenGL and DirectX expect triangles to be declared in counter-clockwise order. We will also use counter-clockwise winding. Now let's see how ordering impacts the edge function.

Why does winding matter when it comes to the edge function? You may have noticed that since the beginning of this chapter, in all figures we have drawn the triangle vertices in clockwise order. We have also defined the edge function as:

$$
\begin{array}{l}
E_{AB}(P) &=& (P.x - A.x) * (B.y - A.y) - \\
&& (P.y - A.y) * (B.x - A.x)
\end{array}
$$

If we respect this convention, then points to the right of the line defined by the vertices A and B will be positive. For example, a point to, the right of V0V1, V1V2, or V2V0 would be positive. However, if we were to declare the vertices in counter-clockwise order, points to the right of an edge defined by vertices A and B would still be positive, but then they would be outside the triangle. In other words, points overlapping the triangle would not be positive but negative (Figure 10). You can potentially still get the code working with positive numbers with a small change to the edge function:

$$E_{AB}(P) = (A.x - B.x) * (P.y - A.y) - (A.y - B.y) * (P.x - A.x).$$

In conclusion, depending on the ordering convention you use, you may need to use one version of the edge function or the other.

## Barycentric Coordinates

![Figure 11: the area of a parallelogram is twice the area of a triangle.](/images/rasterization/barycentric1.png?)

Computing barycentric coordinates are not necessary to get the rasterization algorithm working. For a naive implementation of the rendering technique, all you need is to project the vertices and use a technique like an edge function that we described above, to find if pixels are inside triangles. These are the only two necessary steps to produce an image. However, the result of the edge function which as we explained above, can be interpreted as the area of the parallelogram defined by vectors A and B can directly be used to compute these barycentric coordinates. Thus, it makes sense to study the edge function and the barycentric coordinates at the same time.

Before we get any further though, let's explain what these barycentric coordinates are. First, they come in a set of <b>three floating point numbers</b> which in this lesson, we will denote \(\lambda_0\), \(\lambda_1\) and \(\lambda_2\). Many different conventions exist but Wikipedia uses the greek letter lambda as well (\(\lambda\)) which is also used by other authors (the greek letter omega \(\omega\) is also sometimes used). This doesn't matter, you can call them the way you want. In short, the coordinates can be used to define any point on the triangle in the following manner:

$$P = \lambda_0 * V0 + \lambda_1 * V1 + \lambda_2 * V2.$$ 

Where as usual, V0, V1, and V2 are the vertices of a triangle. These coordinates can take on any value, but for points that are inside the triangle (or lying on one of its edges) they can only be in the range [0,1] and the sum of the three coordinates is equal to 1. In other words:

$$\lambda_0 + \lambda_1 + \lambda_2 = 1, \text{ for } P \in \triangle{V0, V1, V2}.$$

![Figure 12: how do we find the color of P?](/images/rasterization/barycentric2.png?)

This is a form of interpolation if you want. They are also sometimes defined as **weights** for the triangle's vertices (which is why in the code we will denote them with the letter w). A point overlapping the triangle can be defined as "a little bit of V0 plus a little bit of V1 plus a little bit of V2". Note that when any of the coordinates is 1 (which means that the others in this case are necessarily 0) then the point P is equal to one of the triangle's vertices. For instance if \(\lambda_2 = 1\) then P is equal to V2. Interpolating the triangle's vertices to find the position of a point inside the triangle is not that useful. But the method can also be used to interpolate across the surface of the triangle any quantity or variable that has been defined at the triangle's vertices. Imagine for instance that you have defined a color at each vertex of the triangle. Say V0 is red, V1 is green and V2 is blue (Figure 12). What you want to do, is find how these three colors interpolated across the surface of the triangle. If you know the barycentric coordinates of a point P on the triangle, then its color \(C_P\) (which is a combination of the triangle vertices' colors) is defined as:

$$C_P = \lambda_0 * C_{V0} + \lambda_1 * C_{V1} + \lambda_2 * C_{V2}.$$ 

This is a very handy technique that is going to be useful to shade triangles. Data associated with the vertices of triangles is called **vertex attribute**. This is a very common and very important technique in CG. **The most common vertex attributes are colors, normals, and texture coordinates**. What this means in practice, is that generally when you define a triangle you don't only pass on to the renderer the triangle vertices but also its associated vertex attributes. For example, if you want to shade the triangle you may need color and normal vertex attribute, which means that each triangle will be defined by 3 points (the triangle vertex positions), 3 colors (the color of the triangle vertices), and 3 normals (the normal of the triangle vertices). Normals too can be interpolated across the surface of the triangle. Interpolated normals are used in a technique called [**smooth shading**](http://en.wikipedia.org/wiki/Gouraud_shading) which was first introduced by Henri Gouraud. We will explain this technique later when we get to shading.

How do we find these barycentric coordinates? It turns out to be simple. As mentioned above when we presented the edge function, the result of the edge function can be interpreted as the area of the parallelogram defined by the vectors A and B. If you look at Figure 8, you can easily see that the area of the triangle defined by the vertices V0, V1, and V2, is just half of the area of the parallelogram defined by the vectors A and B. The area of the triangle is thus half the area of the parallelogram which we know can be computed by the cross-product of the two 2D vectors A and B:

$$Area_{\triangle{V0V1V2}}= {1 \over 2} {A \times B} = {1 \over 2}(A.x * B.y - A.y * B.x).$$

![Figure 13: connecting P to each vertex of the triangle forms three sub-triangles.](/images/rasterization/barycentric3.png?)

If the point P is inside the triangle, then you can see by looking at Figure 3, that we can draw three sub-triangles: V0-V1-P (green), V1-V2-P (magenta), and V2-V0-P (cyan). It is quite obvious that the sum of these three sub-triangle areas, is equal to the area of the triangle V0-V1-V2:

$$
\begin{array}{l}
Area_{\triangle{V0V1V2}} =&Area_{\triangle{V0V1P}} + \\& Area_{\triangle{V1V2P}} + \\& Area_{\triangle{V2V0P}}.
\end{array}
$$

![Figure 14: the values for \(\lambda_0\), \(\lambda_1\) and \(\lambda_2\) depends on the position of P on the triangle.](/images/rasterization/barycentric4.png?)

Let's first try to intuitively get a sense of how they work. This will be easier hopefully if you look at Figure 14. Each image in the series shows what happens to the sub-triangle as a point P which is originally on the edge defined by the vertices V1-V2, moves towards V0. In the beginning, P lies exactly on the edge V1-V2. In a way, this is similar to a basic linear interpolation between two points. In other words, we could write:

$$P = \lambda_1 * V1 + \lambda_2 * V2$$

With \(\lambda_1 + \lambda_2 = 1\) thus \(\lambda_2 = 1 - \lambda_1\). What's more interesting in this particular case is that if the generic equation for computing the position of P using barycentric coordinates is:

$$P = \lambda_0 * V0 + \lambda_1 * V1 + \lambda_2 * V2.$$ 

Thus, it clearly shows that in this particular case, \(\lambda_0\) is equal to 0.

$$
\begin{array}{l}
P = \lambda_0 * V0 + \lambda_1 * V1 + \lambda_2 * V2,\\
P = 0 * V0 + \lambda_1 * V1 + \lambda_2 * V2,\\
P = \lambda_1 * V1 + \lambda_2 * V2.
\end{array}
$$ 

This is pretty simple. Note also that in the first image, the red triangle is not visible. Note also that P is closer to V1 than it is to V2. Thus, somehow, \(\lambda_1\) is necessarily greater than \(\lambda_2\). Note also that in the first image, the green triangle is bigger than the blue triangle. So if we summarize: when the red triangle is not visible, \(\lambda_0\) is equal to 0. \(\lambda_1\) is greater than \(\lambda_2\) and the green triangle is bigger than the blue triangle. Thus somehow, there seems to be a relationship between the area of the triangles and the barycentric coordinates. Furthermore, the red triangle seems associated with \(\lambda_0\) the green triangle with \(\lambda_1\), and the blue triangle with \(\lambda_2\).

- \(\lambda_0\) is proportional to the area of the red triangle,
- \(\lambda_1\) is proportional to the area of the green triangle,
- \(\lambda_2\) is proportional to the area of the blue triangle.

Now, let's jump directly to the last image. In this case, P is equal to V0. This is only possible if \(\lambda_0\) is equal to 1 and the two other coordinates are equal to 0:

$$
\begin{array}{l}
P = \lambda_0 * V0 + \lambda_1 * V1 + \lambda_2 * V2,\\
P = 1 * V0 + 0 * V1 + 0 * V2,\\
P = V0.
\end{array}
$$

![Figure 15: to compute one of the barycentric coordinates, use the area of the triangle defined by P and the edge opposite to the vertex for which the barycentric coordinate needs to be computed.](/images/rasterization/barycentric5.png?)

Note also that in this particular case, the blue and green triangles have disappeared and that the area of the triangle V0-V1-V2 is the same as the area of the red triangle. This confirms our intuition that there is a relationship between the area of the sub-triangles and the barycentric coordinates. Finally, from the above observation we can also say that each barycentric coordinate is somehow related to the area of the sub-triangle defined by the edge directly opposite to the vertex the barycentric coordinate is associated with, and the point P. In other words (Figure 15):

- \(\color{red}{\lambda_0}\) is associated with V0. The edge opposite V0 is V1-V2. V1-V2-P defines the red triangle.
- \(\color{green}{\lambda_1}\) is associated with V1. The edge opposite V1 is V2-V0. V2-V0-P defines the green triangle.
- \(\color{blue}{\lambda_2}\) is associated with V2. The edge opposite V2 is V0-V1. V0-V1-P defines the blue triangle.

If you haven't noticed yet, the area of the red, green, and blue triangles are given by the respective edge functions that we have been using before to find if P is inside the triangle, divided by 2 (remember that the edge function itself gives the "signed" area of the parallelogram defined by the two vectors A and B, where A and B can be any of the three edges of the triangle):

$$
\begin{array}{l}
\color{red}{Area_{tri}(V1,V2,P)}=&{1\over2}E_{12}(P),\\
\color{green}{Area_{tri}(V2,V0,P)}=&{1\over2}E_{20}(P),\\
\color{blue}{Area_{tri}(V0,V1,P)}=&{1\over2}E_{01}(P).\\
\end{array}
$$

The barycentric coordinates can be computed as the ratio between the area of the sub-triangles and the area of the triangle V0V1V2:

$$\begin{array}{l}
\color{red}{\lambda_0 = \dfrac{Area(V1,V2,P) } {Area(V0,V1,V2)}},\\
\color{green}{\lambda_1 = \dfrac{Area(V2,V0,P)}{Area(V0,V1,V2)}},\\
\color{blue}{\lambda_2 = \dfrac{Area(V0,V1,P)}{Area(V0,V1,V2)}}.\\
\end{array}
$$

What the division by the triangle area does, essentially normalizes the coordinates. For example, when P has the same position as V0, then the area of the triangle V2V1P (the red triangle) is the same as the area of the triangle V0V1V2. Thus dividing one by the over gives 1, which is the value of the coordinate \(\lambda_0\). Since in this case, the green and blue triangles have area 0, \(\lambda_1\) and \(\lambda_2\) are equal to 0 and we get:

$$P = 1 * V0 + 0 * V1 + 0 * V2 = V0.$$

Which is what we expect.

To compute the area of a triangle we can use the edge function as mentioned before. This works for the sub-triangles as well as the main triangle V0V1V2. However the edge function returns the area of the parallelogram instead of the area of the triangle (Figure 8) but since the barycentric coordinates are computed as the ratio between the sub-triangle area and the main triangle area, we can ignore the division by 2 (this division which is in the numerator and the denominator cancel out):

$$\lambda_0 = \dfrac{Area_{tri}(V1,V2,P)}{Area_{tri}(V0,V1,V2)} = \dfrac{1/2 E_{12}(P)}{1/2E_{12}(V0)} =  \dfrac{E_{12}(P)}{E_{12}(V0)}.$$

Note that: \( E_{01}(V2) = E_{12}(V0) = E_{20}(V1) = 2 * Area_{tri}(V0,V1,V2)\).

Let's see how it looks in the code. We were already computing the edge functions before to test if points were inside triangles. Only, in our previous implementation, we were just returning true or false depending on whether the result of the function was either positive or negative. To compute the barycentric coordinates, we need the actual result of the edge function. We can also use the edge function to compute the area (multiplied by 2) of the triangle. Here is a version of an implementation that tests if a point P is inside a triangle and if so, computes its barycentric coordinates:

```
float edgeFunction(const Vec2f &a, const Vec3f &b, const Vec2f &c)
{
    return (c.x - a.x) * (b.y - a.y) - (c.y - a.y) * (b.x - a.x);
}

float area = edgeFunction(v0, v1, v2); // area of the triangle multiplied by 2
float w0 = edgeFunction(v1, v2, p); // signed area of the triangle v1v2p multiplied by 2
float w1 = edgeFunction(v2, v0, p); // signed area of the triangle v2v0p multiplied by 2
float w2 = edgeFunction(v0, v1, p); // signed area of the triangle v0v1p multiplied by 2

// if point p is inside triangles defined by vertices v0, v1, v2
if (w0 >= 0 && w1 >= 0 && w2 >= 0) {
    // barycentric coordinates are the areas of the sub-triangles divided by the area of the main triangle
    w0 /= area;
    w1 /= area;
    w2 /= area;
}
```

Let's try this code to produce an actual image.

<details>
We know that:

$$\lambda_0 + \lambda_1 + \lambda_2 = 1.$$

We also know that we can compute any value across the surface of the triangle using the following equation:

$$Z = \lambda_0 * Z0 + \lambda_1 * Z1 + \lambda_0 * Z2.$$

The value that we interpolate in this case is Z which can be anything we want or as the name suggests, the z-coordinate of the triangle's vertices in camera space. We can re-write the first equation:

$$\lambda_0 = 1 - \lambda_1 - \lambda_2.$$

If we plug this equation in the equation to compute Z and simplify, we get:

$$Z = Z0 + \lambda_1(Z1 - Z0) + \lambda_2(Z2 - Z0).$$

\(Z1 - Z0\) and \(Z2 - Z0\) can generally be precomputed which simplifies the computation of Z to two additions and two multiplications. We mention this optimization because GPUs use it and people may mention it for this reason essentially.
</details>

## Interpolate vs. Extrapolate

![Figure 16: interpolation vs. extrapolation.](/images/rasterization/extrapolate.png?)

One thing worth noticing is that the computation of barycentric coordinates works independently from its position with respect to the triangle. In other words, the coordinates are valid if the point is inside our outside the triangle. When the point is inside, using the barycentric coordinates to evaluate the value of a vertex attribute is called interpolation, and when the point is outside, we speak of extrapolation. This is an important detail because in some cases, we will have to evaluate the value of a given vertex attribute for points that potentially don't overlap triangles. To be more specific, this will be needed to compute the derivatives of the triangle texture coordinates for example. These derivatives are used to filter textures properly. If you are interested in learning more about this particular topic we invite you to read the lesson on Texture Mapping. In the meantime, all you need to remember is that barycentric coordinates are valid even when the point doesn't cover the triangle. You also need to know about the difference between vertex attribute extrapolation and interpolation.

## Rasterization Rules

![Figure 17: pixels may cover an edge shared by two triangles.](/images/rasterization/top-left.png?)

![Figure 18: if the geometry is semi-transparent, a dark edge may appear where pixels overlap the two triangles.](/images/rasterization/top-left2.png?)

![Figure 19: top and left edges.](/images/rasterization/top-left3.png?)

In some special cases, a pixel may overlap more than one triangle. This happens when a pixel lies exactly on an edge shared by two triangles as shown in Figure 17. Such a pixel would pass the coverage test for both triangles. If they are semi-transparent, a dark edge may appear where the pixels overlap the two triangles as a result of the way semi-transparent objects are combined (imagine two super-imposed semi-transparent sheets of plastic. The surface is more opaque and looks darker than the individual sheets). You would get something similar to what you can see in Figure 18, which is a darker line where the two triangles share an edge.

The solution to this problem is to come up with some sort of rule that guarantees that a pixel can never overlap twice two triangles sharing an edge. How do we do that? Most graphics APIs such as OpenGL and DirectX define something which they call the **top-left rule**. We already know the coverage test returns true if a point is either inside the triangle or if it lies on any of the triangle edges. What the top-left rule says though, is that the pixel or point is considered to overlap a triangle if it is either inside the triangle or lies on either a triangle's top edge or any edge that is considered to be a left edge. What is a top and theft edge? If you look at Figure 19, you can easily see what we mean by the top and left edges.

- A top edge is an edge that is perfectly horizontal and whose defining vertices are above the third one. Technically this means that the y-coordinates of the vector V[(X+1)%3]-V[X] are equal to 0 and that its x-coordinates are positive (greater than 0).
- A left edge is essentially an edge that is going up. Keep in mind that in our case, vertices are defined in clockwise order. An edge is considered to go up if its respective vector V[(X+1)%3]-V[X] (where X can either be 0, 1, 2) has a positive y-coordinate.

<details>
Of course, if you are using a counter-clockwise order, a top edge is an edge that is horizontal and whose x-coordinate is negative, and a left edge is an edge whose y-coordinate is negative.
</details>

In pseudo-code we have:

```
// Does it pass the top-left rule?
Vec2f v0 = { ... };
Vec2f v1 = { ... };
Vec2f v2 = { ... };

float w0 = edgeFunction(v1, v2, p); 
float w1 = edgeFunction(v2, v0, p); 
float w2 = edgeFunction(v0, v1, p); 

Vec2f edge0 = v2 - v1;
Vec2f edge1 = v0 - v2;
Vec2f edge2 = v1 - v0;

bool overlaps = true;

// If the point is on the edge, test if it is a top or left edge, 
// otherwise test if  the edge function is positive
overlaps &= (w0 == 0 ? ((edge0.y == 0 && edge0.x > 0) ||  edge0.y > 0) : (w0 > 0));
overlaps &= (w1 == 0 ? ((edge1.y == 0 && edge1.x > 0) ||  edge1.y > 0) : (w1 > 0));
overlaps &= (w1 == 0 ? ((edge2.y == 0 && edge2.x > 0) ||  edge2.y > 0) : (w2 > 0));

if (overlaps) {
    // pixel overlap the triangle
    ...
}
```

This version is valid as a proof of concept but highly unoptimized. The key idea is to first check whether any of the values return by returned function is equal to 0 which means that the point lies on the edge. In this case, we test if the edge in question is a top-left edge. If it is, it returns true. If the value returned by the edge function is not equal to 0, we then return true if the value is greater than 0. We won't implement the top-left rule in the program provided with this lesson.

## Putting Things Together: Finding if a Pixel Overlaps a Triangle

![Figure 20: Example of vertex attribute linear interpolation using barycentric coordinates.](/images/rasterization/raster2d.png?)

Let's test the different techniques we learned about in this chapter, in a program that produces an actual image. We will just assume that we have projected the triangle already (check the last chapter of this lesson for a complete implementation of the rasterization algorithm). We will also assign a color to each vertex of the triangle. Here is how the image is formed. We will loop over all the pixels in the image and test if they overlap the triangle using the edge function method. All three edges of the triangle are tested against the current position of the pixel, and if the edge function returns a positive number for all the edges then the pixel overlaps the triangle. We can then compute the pixel's barycentric coordinates and use these coordinates to shade the pixel by interpolating the color defined at each vertex of the triangle. The result of the frame-buffer is saved to a PPM file (that you can read with Photoshop). The output of the program is shown in Figure 20.

Note that one possible optimization for this program would be to loop over the pixels contained in the bounding box of the triangle. We haven't made this optimization in this version of the program but you can make it yourself if you wish (using the code from the previous chapters). You can also check the source code of this lesson (available in the last chapter).

Note also that in this version of the program, we move point P to the center of each pixel. You could as well use the pixel integer coordinates. You will find more details on this topic in the next chapter.

```
// c++ -o raster2d raster2d.cpp
// (c) www.scratchapixel.com

#include &ltcstdio&gt
#include &ltcstdlib&gt
#include &ltfstream&gt

typedef float Vec2[2];
typedef float Vec3[3];
typedef unsigned char Rgb[3];

inline
float edgeFunction(const Vec2 &a, const Vec2 &b, const Vec2 &c)
{ return (c[0] - a[0]) * (b[1] - a[1]) - (c[1] - a[1]) * (b[0] - a[0]); }

int main(int argc, char **argv)
{
    Vec2 v0 = {491.407, 411.407};
    Vec2 v1 = {148.593, 68.5928};
    Vec2 v2 = {148.593, 411.407};
    Vec3 c0 = {1, 0, 0};
    Vec3 c1 = {0, 1, 0};
    Vec3 c2 = {0, 0, 1};
    
    const uint32_t w = 512;
    const uint32_t h = 512;
    
    Rgb *framebuffer = new Rgb[w * h];
    memset(framebuffer, 0x0, w * h * 3);
    
    float area = edgeFunction(v0, v1, v2);
    
    for (uint32_t j = 0; j &lt h; ++j) {
        for (uint32_t i = 0; i &lt w; ++i) {
            Vec2 p = {i + 0.5f, j + 0.5f};
            float w0 = edgeFunction(v1, v2, p);
            float w1 = edgeFunction(v2, v0, p);
            float w2 = edgeFunction(v0, v1, p);
            if (w0 &gt= 0 && w1 &gt= 0 && w2 &gt= 0) {
                w0 /= area;
                w1 /= area;
                w2 /= area;
                float r = w0 * c0[0] + w1 * c1[0] + w2 * c2[0];
                float g = w0 * c0[1] + w1 * c1[1] + w2 * c2[1];
                float b = w0 * c0[2] + w1 * c1[2] + w2 * c2[2];
                framebuffer[j * w + i][0] = (unsigned char)(r * 255);
                framebuffer[j * w + i][1] = (unsigned char)(g * 255);
                framebuffer[j * w + i][2] = (unsigned char)(b * 255);
            }
        }
    }
    
    std::ofstream ofs;
    ofs.open("./raster2d.ppm");
    ofs << "P6\n" << w << " " << h << "\n255\n";
    ofs.write((char*)framebuffer, w * h * 3);
    ofs.close();
    
    delete [] framebuffer;
    
    return 0; 
}
```

As you can see and in conclusion, we can say that the rasterization algorithm is in itself quite simple (and the basic implementation of this technique is quite easy as well).

## Conclusion and What's Next?

![Figure 21: barycentric coordinates are constant along lines parallel to an edge.](/images/rasterization/barycentric6.png?)

There are many interesting techniques and trivia related to the topic of barycentric coordinates but this lesson is just an introduction to the rasterization algorithm thus we won't go any further. One trivia that is interesting to know though, is that barycentric coordinates are constant along lines parallel to an edge (as shown in Figure 21).

In this lesson, we learned two important methods and various concepts.

- First, we learned about the edge function and how it can be used to find if a point P overlaps a triangle. The edge function is computed for each edge of the triangle, and a second vector is defined by the edge first vertex and another point P. If for all three edges, the function is positive, then point P overlaps the triangle.
- Furthermore, we also learned that the result of the edge function can also be used to compute the barycentric coordinates of point P. These coordinates can be used to interpolate vertex data or vertex attributes across the surface of the triangle. They can be interpreted as weights for the various vertices. The most common vertex attribute is color, normal, and texture coordinates.