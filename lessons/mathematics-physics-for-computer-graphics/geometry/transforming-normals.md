## What Is a Normal?

![Figure 1: the tangent (T) and bi-tangent (B) are lying in the plane tangent at P. Taking the cross product between T and B gives the surface normal N. Note that T, B, and N are orthogonal to each other and form a Cartesian coordinate system.](/images/geometry/normal.png?)

We briefly mentioned what normals were in the first chapter of this lesson. A surface normal from a surface at P is a vector perpendicular to the tangent plane to that surface at P. We will learn more about how to calculate normals as we get to the lessons on geometric primitives. But let's say for now that if you know the tangent T and bi-tangent B of the surface at P (which defines the plane tangent to the surface at P), then we can calculate the surface normal at P using a simple cross-product between T and B:

$$N = T \times B$$

Remember what we have said about the cross-product operation. It is anticommutative, meaning that swapping the position of any two arguments negates the result. In other words: \(T \times B = N\) and \(B \times T = -N\). In practice, it just means that you will have to be careful to calculate the normal so that it points away from the surface (for reasons we will explain when we will get to the lessons on Shading), but we will come back on this again in other lessons.

## Transforming Normals

![Figure 2: a) in blue, we draw the normal to the line AB. b) We have transformed the point AB by scaling them by the factor (2, 1, 0). If we do the same for the normal with coordinates (1, 1, 0), we can see that the transformed normal is not perpendicular anymore to A'B'. c) we transformed the normal by transposing the inverse of the matrix. The resulting normal is orthogonal to A'B'.](/images/geometry/transformnormal1.png?)

Why not consider normals as vectors? Why do we take the pain of differentiating them? In the previous chapters, we learned to use matrix multiplication to transform points and vectors. The problem with normals is that we assume that transforming them the same way we transform points and vectors will work. This is sometimes the case, for example, when the matrix scales the normal uniformly (that is, when the values of the matrix along the diagonal, which we have learned encode the scale values applied to the transformed point or vector, are all the same). But let's now consider the case where a non-uniform scale is applied to an object. Lets draw (in 2D) a line which is passing through the points A=(0, 1, 0) and B=(1, 0, 0) as illustrated in figure 1\. If you draw another line from the origin to the coordinate (1, 1, 0), you can see that this line is perpendicular to our plane. Let's consider this our normal N (technically, we should normalize this vector but not doing so will not be a problem for this explanation). Now let's say that we apply a nonuniform scale to the plane using the following matrix:

$$
M=
\begin{bmatrix}
2&0&0&0\\
0&1&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

This matrix scales the x-coordinate of any point (or vector) by 2 and leaves the other coordinates unchanged. Applied to our example, we get A'=A*M which gives A'=(0, 1, 0), and B'=B*M, which is equal to (2, 0, 0). Similarly, if we calculate N' as N'=N*M, we get N'=(2, 1, 0). Now, if we both draw our new transformed line (going through A' and B') and N', we can see that N' is no longer perpendicular to A'B'. The solution to transforming normals is not to multiply them by the same matrix used for transforming points and vectors but to multiply them by the **transpose of the inverse of that matrix**:

$$N'=N*M^{-1T}$$

Before considering the mathematical proof, let's explain why this solution works using intuition. First, we know that normals represent directions, so a translation does not affect them like vectors. In other words, we can ignore the fourth column and fourth row of our [4x4] matrix and consider the remaining inner, upper-left [3x3] matrix, which encodes the rotation and the scale. We have also explained in this lesson that the transpose of an orthogonal matrix is also its inverse and that rotation matrices are orthogonal. In other words, if Q is an orthogonal matrix, we can write:

\(Q^T=Q^{-1}\) therefore \(Q=Q^{-1T}\)

The transpose of the inverse of an orthogonal matrix Q gives the matrix Q. In other words; this doesn't change anything. Using the transpose of the inverse of that matrix doesn't change the elements from the matrix that encode rotations, and transforming a normal with this transposed inverted matrix, will rotate the normals as if we had used the original matrix (we want the normal to follow any rotation you apply to an object).

<details>
Question from a reader: "But the elements of the matrix \(M\) along the diagonal can encode rotations and scale simultaneously. So if scale and rotations are mixed up in one single matrix, is the matrix still orthogonal?". You would be right if the scaling is different than 1 in any dimension. However, you can see a matrix that encodes both rotations and scaling as a multiplication of two different matrices, one that encodes rotation only \(R\), and one that encodes scaling only \(S\):

$M=R * S$

And the matrix on the left \(R\) would be orthogonal. Therefore saying that the transpose of the inverse of that matrix \(R^{-1T}\) is the same as the matrix itself \(R\) holds. So all we are left to do in our demonstration is to see what happens to the matrix \(S\) when we take the transpose of its inverse.
</details>

The last elements from the matrix we have yet to look at are the numbers along the matrix's diagonal, which encode the scale values. What happens to them when we calculate the transpose of the inverse of a matrix? The transpose operation doesn't change the elements along the diagonal of a matrix. Only the inverse operation changes them. If a point is scaled by a factor of 4, we know that we need to scale it by 0.25 (\(1 \over 4\), the inverse of the original scale factor) to bring it back to its original position. Similarly, the inverse of a scale matrix can easily be calculated by taking the inverse of the scale factors. Applied to our example, we get the following:

$$
M^{-1T}=
\begin{bmatrix}
1 \over 2&0&0&0\\
0&1&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}
$$

If we apply this matrix to our normal N=(1, 1, 0) we get N'=(0.5, 1, 0). Let's now draw this vector next to line A'B' and check that it is perpendicular to the line (figure 2c). As you can see, we now have a normal orthogonal to the transformed line A'B'.

<details>
It is also possible to calculate the normals from transformed vertices, but this technique can't be used, for example, with quadratic shapes. Imagine a sphere rendered as a quadradic shape. If you scale the sphere along the x-axis by 2, you will get an ellipsoid. Try to visually imagine what's happening to the normal of a sphere transformed that way if you apply the original matrix to the normals. Suppose you can calculate the derivatives of a point on the surface(the tangent and bitangent). In that case, you can calculate a transformed normal from these transformed derivatives, no matter what type of geometric primitive you are dealing with. This is the technique we will be using in our basic renderer, but we will only sometimes have access to these derivatives, so using the transpose of the inverse matrix is still the only valid technique we can use in these cases.
</details>

Here is the mathematical proof that the transpose of the inverse is what we need to transform normals. Remember that the dot product of two orthogonal vectors is equal to 0\. Note also that we can re-write the dot product as a matrix multiplication between a [1x3] and a [3x1] matrix, which gives us a [1x1] matrix, one number, as with the result of the dot product. If the result of the dot product is 0, then the result of the matrix multiplication (assuming you are using the same vectors) should also be 0\. Imagine that we have two vectors orthogonal to each other at point P. One vector is \(v\) and lies within the plane tangent to P, and \(n\) is the normal at P. The dot product of v and n is 0 since n is the normal and v lies in a plane tangent to P. We can also re-write n as a [3x1] matrix, which we can get by transposing n itself, and multiply \(v\) as a [1x3] matrix by \(n^T\), which result should also be 0 (since the formula of the matrix multiplication is the same as the formula of a dot product in that case):

$$
v \cdot n = 
\begin{pmatrix}
v_x & v_y & v_x
\end{pmatrix}
*
\begin{pmatrix}
n_x\\n_y\\n_z
\end{pmatrix}
=v * n^T=0
$$

$$v \cdot n = v * n^T = v_x * n_x + v_y * n_y + v_z * n_z$$

We can also write:

$$v * n^T = v * M * M^{-1} * n^T = v * I * n^T$$

where \(M\) is a matrix we want to transform P with and \(I\) is the identity matrix. We know that the multiplication of a matrix with its inverse gives the identity matrix, so in essence, technically, the term \(M^{-1} * M\) we added in the middle of the term \(v * n^T\) does nothing. However, let's see what we can do by re-arranging and re-writing the terms:

$$v * n^T = (v*M) * (n*M^{-1T})^T$$

First, we can notice that the first term on the left, \(v*M\), is nothing else than the vector \(v'\) which is the vector \(v\) transformed by the matrix \(M\). We said before that transforming vectors with the matrix doesn't work for normals, but it does work for vectors lying in the plane tangent to P. In other words:

$$v' = v * M$$

The second term on the right has been re-arranged. We moved the matrix \(M^{-1}\) to the right of \(n^T\). This is possible only if we transpose the matrix itself, which is why we wrote \(M^{-1T}\). Remember that \(A \times B = B^T \times A\). Finally, we can write:

$$v * n^T =v' * n'^T$$

This equality has to be true because the dot product between v and n should be the same after the two vectors have been transformed (the dot product is invariant under linear transformation). Thus, if \((n * M^{-1T})^T=n'^T\), then \(n'=n * M^{-1T}\).