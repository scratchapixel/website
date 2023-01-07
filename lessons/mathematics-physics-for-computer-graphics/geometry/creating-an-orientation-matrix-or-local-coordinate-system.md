## Creating an Orientation Matrix or Local Coordinate System

In this chapter, we will use what we have learned so far about coordinate systems and what they represent to build a local coordinate system (or frame) from a vector which can also be a normal. This technique is often used in the rendering pipeline to convert points and vectors defined in one coordinate system to another. The idea is to let the normal at that point become one of the axes of that local coordinate system (often aligned with the up vector and let the tangent and bi-tangent of that point become the other two orthogonal axes of that local frame.

![Figure 1: the tangent (T) and bi-tangent (B) are lying in the plane tangent at P. Taking the cross product between T and B gives the surface normal N. Note that T, B, and N are orthogonal to each other and form a Cartesian coordinate system.](/images/geometry/normal.png?)

The best way of constructing such as local frame is to use the **normal**, the **tangent**, and **bi-tangent** at the surface P, which, as we explained before, lie in the plane tangent to P at the surface. The three axes should be orthogonal and of unit length. In the lessons related to computing the intersection between a ray and various geometric primitives, we will usually also learn how to calculate the **derivatives** at the hit point (which we will call **dPdu** and **dPdv**) which are the technical terms used to describe the tangent and bitangent at P (check the lesson on geometric primitives to learn more about derivatives). We usually find the normal at P from a cross-product between dPdu and dPdv. However, you will have to be careful about the direction in which these two vectors are pointing to be sure that the result of this cross-product is a vector oriented away from the surface (and not inward). If you know the directions the two vectors point to in space, then you can use the right-hand rule to figure out the order you should use them in to get a normal that points in the right direction (see chapter 3 [Math Operations on Points and Vectors](/lessons/mathematics-physics-for-computer-graphics/geometry/math-operations-on-points-and-vectors)).

![Figure 2: on the left, the cross product of A and B gives a vector C point away from the normal. On the right, the cross product of B and A provides a vector that points inwards. The direction of the resulting vector can easily be found using the right-hand rules.](/images/geometry/crossnormal.png?)

Assuming the normal N will correspond to the up vector, the tangent will be aligned with the right vector, and the bitangent aligned along the dow vector, we can write these tree vectors as the rows of the following [4x4] matrix:

$$
\begin{bmatrix}
T_x&T_y&T_z&0\\
N_x&N_y&N_z&0\\
B_x&B_y&B_z&0\\
0&0&0&1
\end{bmatrix}
$$

![Figure 3: the order in which the axis coordinates are written as rows of the orientation matrix depends on the convention you will use. The up vector is usually the y-axis, but in shading, it's common to choose the z-axis as the up-vector. If we assume a right-hand coordinate system, the z-axis in the figure on the left (a) would have to be the tangent, and the x-axis would have to be the bi-tangent. In Figure b, y would have to be the tangent and x the bitangent. Point the index finger along the tangent and the middle finger along the bi-tangent to find out the direction of normal using the right-hand mnemonic technique.](/images/geometry/normal2.png?)

However, you will have to be careful about the context (in the code) in which you will use this matrix. For example, some parts of the code might be using a different convention where the up vector is considered the z-axis. This is usually particularly true for code that deals with shading tasks (to understand why to check the previous chapter on [Spherical Coordinates](/lessons/mathematics-physics-for-computer-graphics/geometry/spherical-coordinates-and-trigonometric-functions)). In which case, the rows should be re-ordered in the following way:

$$
\begin{bmatrix}
T_x&T_y&T_z&0\\
B_x&B_y&B_z&0\\
N_x&N_y&N_z&0\\
0&0&0&1
\end{bmatrix}
$$

As you can see, the normal coordinates are now on the third row of the matrix. Why do we suddenly use the convention of aligning the normal to the surface with the z-axis of the coordinate system? It isn't obvious, but sadly, this is also a convention used in most papers related to shading, which we can't ignore for this reason. So it is preferable to follow the same convention. Figure 4 shows how the up vector is defined by the y-vector in the world coordinate system but is represented by the z-vector in the local coordinate system.

<details>
Remember that if you use a column-major order convention (Scratchapixel uses a row-major order convention), the vectors must write as columns, not rows. So, for instance, if the z-vector is considered the up vector, in the first column, you will write the coordinates of T, in the second, the coordinates of B, and in the third, the coordinates of N.
</details>

![Figure 4: it is sometimes helpful to express a vector V in a local coordinate system which we can create from the normal and tangent at a point on the surface. If the normal N and tangent T are known, it is easy to calculate the bitangent B and create a matrix from these three vectors that will represent this world-to-local coordinate system matrix that transforms V in world space to the defined space by N, T, and B. This technique is beneficial in shading. Note that the V vector doesn't change direction. Its coordinates are different in the two coordinate systems.](/images/geometry/localcoord.png?)

So, you may ask now, what do we do with this matrix? Suppose you have a vector \(v\) defined, let's say, in world space (but any other space will do as well). In that case, multiplying this vector by this matrix \(M\) will give a vector \(v_M\) whose coordinates are defined in regards to the local coordinate system you constructed from \(N\), \(T\) and \(B\). As you can see, there is no translation value set on the fourth row of the matrix, which is why we call that type of matrix an **orientation matrix**. This is because you only want to use this matrix with vectors. It will mainly be used in shading, where expressing vectors coordinates in relation to the surface normal (where N is usually aligned along the up vector, which is either by convention the y- or z-axis) can significantly simplify the computation involved in finding out the color of an object at the point of intersection with the ray (Figure 4). This technique will be studied in detail in the lesson on Shading.

<details>
**Affine Space**: some renderers (such as [Embree from Intel](http://software.intel.com/en-us/articles/embree-photo-realistic-ray-tracing-kernels)) prefer to represent matrices or transformation as [affine space](http://en.wikipedia.org/wiki/Affine_space) in which a Cartesian coordinate system is defined as a location in space (the origin of the coordinate system say O for instance) and three axes (Vx, Vy, Vz). 
</details>