## Conventions. A Word of Warning!

You may be surprised to find that the information we give on this page differs from what you see in other books or on the internet. The information is the same, but the order or the sign of the matrix coefficients may differ. This is because different authors/programs use different conventions. Try to follow the logic of this lesson without paying too much attention to what other documents might say, and read the next chapter, which will explain exactly how different conventions change how we present them on paper and implement them in a program.

## Point-Matrix Multiplication

In this lesson, we will start to put all the things we have learned about points, vectors, matrices, and coordinate systems together. And at last, you will learn how matrices work. We mentioned in the previous chapter that two matrices needed compatible sizes to be multiplied by each other. For instance, the matrices of size **m x p** and **p x n** can be multiplied by each other. We also mentioned in the previous chapter that we would primarily deal with 4x4 matrices in computer graphics.

A point or a vector is a sequence of three numbers, and for this reason, they too can be written as a 1x3 matrix, a matrix that has one row and three columns. Here is an example of a point written in matrix form:

$$P = [x y z].$$

The trick here is that if we can write points and vectors as [1x3] matrices, we can multiply them by other matrices. Remember that the matrix **m x p** can be multiplied by the matrix **p x n** to give the matrix **m x n**. If the first matrix is a point, we can write m = 1 and p = 3. This implies that the **p x n** matrix is something of form 3 x n where n can be any number greater than 1. In theory, a multiplication of a [1x3] matrix by any of the following matrices would work: [3x1], [3x2], [3x3], [3x4], etc. Here is an example of a [1x3]*[3x4] matrix multiplication:

$$
\begin{bmatrix}x & y & z\end{bmatrix} *
\begin{bmatrix}
c_{00}&c_{01}&{c_{02}}&c_{03}\\
c_{10}&c_{11}&{c_{12}}&c_{13}\\
c_{20}&c_{21}&{c_{22}}&c_{23}\\
\end{bmatrix}
$$

There are two things we need to remember now to make sense of what we will explain. First, a point multiplied by a matrix transforms the point to a new position. Therefore, the result of a point multiplied by a matrix has to be a point. If it weren't the case, we wouldn't be using matrices as a convenient way of transforming points. We must remember that a **m x p** matrix multiplied by a **p x n** matrix gives a **m x n** matrix. If we look at our point as a 1x3 matrix, we need the multiplication result to be another point, a 1x3 matrix. It, therefore, requires the matrix that we will be multiplying the point with to be a 3x3 matrix. Multiplying a 1x3 matrix by a 3x3 matrix gives, as expected, a 1x3 matrix which is another point. Here is what this multiplication looks like:

$$
\begin{bmatrix}x & y & z\end{bmatrix} *
\begin{bmatrix}
c_{00}&c_{01}&{c_{02}}\\
c_{10}&c_{11}&{c_{12}}\\
c_{20}&c_{21}&{c_{22}}\\
\end{bmatrix}
$$

In CG, we usually use 4x4 matrices instead of 3x3 matrices, and we will soon explain why, but for now, let's stick with the 3x3 matrices for a while. To finish this section of the chapter, we will write some pseudocode to show how we can multiply a point \(P\) (or a vector) in its matrix form to a 3x3 matrix to get a newly transformed point \(P_T\). If you need a refresher on matrix multiplication, read the previous chapter. Remember that for each coefficient of the new matrix, you need to multiply each coefficient of the left-hand side matrix's current row with its "equivalent" coefficient of the right-hand side matrix's current column, and sum the resulting products. In pseudocode, it gives something like that (we will provide the version for 4x4 matrices later):

```
// multiply coeffs from row 1 with coeffs from column 1
Ptransformed.x = P.x * c00 + P.y * c10 + P.z * c20
// multiply coeffs from row 1 with coeffs from column 2
Ptransformed.y = P.x * c01 + P.y * c11 + P.z * c21
// multiply coeffs from row 1 with coeffs from column 3
Ptransformed.z = P.x * c02 + P.y * c12 + P.z * c22
```

## The Identity Matrix

The **identity matrix** or **unit matrix** is a square matrix whose coefficients are all 0 except the coefficients along the diagonal, which are set to 1:

$$
\begin{bmatrix}
\color{red}{1} & 0 & 0 \\
0 & \color{red}{1} & 0 \\
0 & 0 & \color{red}{1}
\end{bmatrix}
$$

The result of P multiplied by the identity matrix is P. If we replace the coefficient of the identity matrix in the point-matrix multiplication code, we can clearly understand why:

```
// multiplying P by the identity matrix gives P
Ptransformed.x = P.x * 1 + P.y * 0 + P.z * 0 = P.x
Ptransformed.y = P.x * 0 + P.y * 1 + P.z * 0 = P.y
Ptransformed.z = P.x * 0 + P.y * 0 + P.z * 1 = P.z
```

## The Scaling Matrix

If you look at the code of the point-matrix multiplication, you can see that the coordinates of the point P are respectively multiplied by the coefficients \(R_{00}\) for x, \(R_{11}\) for y and \(R_{22}\) for z. When these coefficients are set to 1 (and all the other coefficients of the matrix are set to 0), we get the identity matrix. However, when these coefficients (along the diagonal) are different than 1 (whether smaller or bigger than 1), then they act as a multiplier on the point's coordinates (in other words, the coordinates of the point are scaled up or down by some amount). If you remember what we have said in the chapter on coordinate systems, multiplying the coordinates of a point by some real numbers results in scaling the point's coordinates. The scaling matrix can therefore be written as:

$$
\begin{bmatrix}
\color{red}{S_X} & 0 & 0 \\
0 & \color{red}{S_Y} & 0 \\
0 & 0 & \color{red}{S_Z}
\end{bmatrix}
$$

Where the real numbers \(S_X\), \(S_Y\) and \(S_Z\) are the scaling factors.

```
// multiplying P by the scaling matrix
Ptransformed.x = P.x * Sx + P.y * 0  + P.z * 0  = P.x * Sx
Ptransformed.y = P.x * 0  + P.y * Sy + P.z * 0  = P.y * Sy
Ptransformed.z = P.x * 0  + P.y * 0  + P.z * Sz = P.z * Sz
```

For example, imagine a point P with coordinates (1, 2, 3). Then, if we set the coefficients of the scaling matrix with Sx = 1, Sy = 2, and Sz = 3, P multiplied by this matrix gives another point whose coordinates are (1, 4, 9).

Note that if either one of the scaling coefficients in the matrix is negative, then the point's coordinate for the corresponding axis will be flipped (it will be mirrored to the other side of the axis).

## The Rotation Matrix

In this paragraph, we will discuss building a matrix that will rotate a point or a vector around one axis of the cartesian coordinate system. And To do so, we will need to use trigonometric functions.

![Figure 1: a 90 degrees counterclockwise rotation.](/images/geometry/rotation.png?)

Let's take a point P defined in a three-dimensional coordinate system with coordinates (1, 0, 0). Let's ignore the z-axis for a while and assume that the point lies in the xy plane. What we want is to transform the point from \(P\) to \(P_T\) by the mean of a rotation (we could do this with a translation, but using a rotation will make our demonstration easier). \(P_T\) coordinates are (0, 1, 0). Figure 1 shows this can be done by rotating the point around the z-axis by 90 degrees **counterclockwise**. Let's assume that we have a matrix \(R\). When \(P\) is multiplied by \(R\) it transforms \(P\) to \(P_T\). Considering what we know about matrix multiplication, let's see how we can re-write a point-matrix multiplication and isolate the computation of each of the transformed point coordinates:

$$
\begin{array}{l}
P_T.x = P.x * R_{00} + P.y * R_{10} + P.z * R_{20}\\
P_T.y = P.x * R_{01} + P.y * R_{11} + P.z * R_{21}\\
P_T.z = P.x * R_{02} + P.y * R_{12} + P.z * R_{22}\\
\end{array}
$$

![Figure 2: a 45 degrees counterclockwise rotation.](/images/geometry/rotation45.png?)

As we said, we don't care much about \(P_T.z\) for now, representing the z-coordinate of \(P_T\). Lets concentrate instead on \(P_T.x\) and \(P_T.y\) which represent respectively the x and y coordinates of \(P_T\). From \(P\) to \(P_T\), the x-coordinate goes from 1 to 0\. If we look at the first line of the code we wrote to compute \(P_T\), it means that \(R_{00}\) has to be equal to 0\. Considering that \(P.y\) and \(P.z\) are 0 anyway, we don't care so much about the values that \(R_{10}\) and \(R_{20}\) may have for now. From \(P\) to \(P_T\) the y-coordinate goes from 0 to 1\. Let's have a look at the second line of code. What do we know about \(P\)? We know that \(P.x\) is 1 and all the other coordinates of \(P\) are 0\. Which necessarily means that \(R_{01}\) has to be 1\. Let's recap. We know that \(R_{00}\) is 0 and \(R_{01}\) is 1\. Let's write it down and see what \(R\) looks like (compare this matrix with the identity matrix):

$$
R_z=
\begin{bmatrix}
0 & 1 & 0 \\
1 & 0 & 0 \\
0 & 0 & 1 \\
\end{bmatrix}
$$

Don't worry for now if you need to understand why the coefficients have the value they have. That will be explained soon. All you want to see is that if you use this matrix to transform \(P\) = (1, 0, 0), you will get \(P_T\) = (0, 1, 0).

$$
\begin{array}{l}
P_T.x = P.x * 0 + P.y * 1 + P.z * 0 = 0\\
P_T.y = P.x * 1 + P.y * 0 + P.z * 0 = 1\\
P_T.z = P.x * 0 + P.y * 0 + P.z * 1 = 0\\
\end{array}
$$

![Figure 3: cosine and sine can be used to determine the coordinate of a point on the x- and y-axis of the unit circle.](/images/geometry/unitcircle.png?)

This is where our knowledge of trigonometric functions will become handy. For example, if we look at a point on the unit circle, we know that its x and y coordinates can be calculated using the sine and the cosine of the angle \(\theta\) (see figure 3).

$$
\begin{array}{l}
x = \cos(\theta) = 0\\
y = \sin(\theta) = 1\\
\text{with } {\theta = {\pi \over 2}}\\
\end{array}
$$

When \(\theta\) = 0, x = 1 and y = 0\. When \(\theta\) = 90 degrees (or \(\pi \over 2\)), x = 0 and y = 1\. That is interesting because you will notice that x = 0 and y = 1 are the values of \(R_{00}\)/\(R_{11}\) and \(R_{01}\)/\(R_{10}\) respectively. So we could re-write the matrix R as:

$$R_z(\theta)=
\begin{bmatrix}
\cos(\theta) & \sin(\theta) & 0 \\
\sin(\theta) & \cos(\theta) & 0 \\
0 & 0 & 1 \\
\end{bmatrix}
=
\begin{bmatrix}
0 & 1 & 0 \\
1 & 0 & 0 \\
0 & 0 & 1 \\
\end{bmatrix} \text{ with } {\theta = {\pi \over 2}}
$$

If you only want to make a rotation of 45 degrees (replace 90 by 45 or \(\pi \over 4\)) and apply \(R\) to \(P\), you will get the coordinates (0.7071, 0.7071) for \(P_T\) which is correct (figure 2). Thus, it seems that we can generalize the notation for R (a matrix that rotates points around the z-axis) and write:

$$
R_z(\theta)=
\begin{bmatrix}
\cos(\theta) & \sin(\theta) & 0 \\
\sin(\theta) & \cos(\theta) & 0 \\
0 & 0 & 1 \\
\end{bmatrix}
$$

We know that the transformation from \(P\) to \(P_T\) works with \(R\) in its current form, but lets now imagine that \(P\) is (0, 1, 0) and \(P_T\) is (1, 0, 0) which is a rotation of 90 degrees but this time **clockwise** (figure 4, see further down). Would \(R\) work and transform \(P\) to \(P_T\)? Lets check:

$$
R_z=
\begin{bmatrix}
\cos(-{\pi \over 2}) & \sin(-{\pi \over 2}) & 0 \\
\sin(-{\pi \over 2}) & \cos(-{\pi \over 2}) & 0 \\
0 & 0 & 1 \\
\end{bmatrix}=
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 0 & 0 \\
0 & 0 & 1 \\
\end{bmatrix}
$$

$$
\begin{array}{lll}
P_T.x = &0 * R_{00} &+& 1 * R_{10} &+& P.z * R_{20} &= \\
&0*0 &+& 1*-1 &+& 0*0&=-1\\
P_T.y = &0 * R_{01} &+& 1 * R_{11} &+& P.z * R_{21} &= \\
&0*-1 &+& 1*0 &+& 0*0&= 0\\
P_T.z = &0 * R_{02} &+& 1 * R_{12} &+& P.z * R_{22} &= \\
&0*0 &+& 1*0 &+& 0*1&= 0\\
\end{array}
$$

That seems like it needs to be corrected since we started from the point with coordinate (0, 1, 0), and after transformation, we have the coordinates (-1, 0, 0) instead of (1, 0, 0). So if we want the coordinates (1, 0, 0), \(R_{01}\) should be 1 (and not -1). So, in that case, we would get for R:

$$
R_z=
\begin{bmatrix}
\cos(-{\pi \over 2}) & -\sin(-{\pi \over 2}) & 0 \\
\sin(-{\pi \over 2}) & \cos(-{\pi \over 2}) & 0 \\
0 & 0 & 1 \\
\end{bmatrix}=
\begin{bmatrix}
0 & 1 & 0 \\ 1 & 0 & 0 \\
0 & 0 & 1 \\
\end{bmatrix}
$$

$$
\begin{array}{lll}
P_T.x = &0 * R_{00} &+& 1 * R_{10} &+& P.z * R_{20} &= \\
&0*0 &+& 1*1 &+& 0*0&=1\\
P_T.y = &0 * R_{01} &+& 1 * R_{11} &+& P.z * R_{21} &= \\
&0*-1 &+& 1*0 &+& 0*0&= 0\\
P_T.z = &0 * R_{02} &+& 1 * R_{12} &+& P.z * R_{22} &= \\
&0*0 &+& 1*0 &+& 0*1&= 0\\
\end{array}
$$

![Figure 4: a 90 degrees clockwise rotation.](/images/geometry/rotationmin90.png?)

We know that the points in the xy plane should stay in the xy plane if we rotate them around the z-axis (so our rotation matrix Rz should not affect the z-coordinate of \(P_T\)). When we look at the code transforming \(P\) to \(P_T\), it is easy to see that the third row and the third column do not affect the computation of \(P_T\). The first two coefficients in the third column, which are used to compute a value for \(P_T.z\), \(R_{02}\) and \(R_{12}\), are set to 0, and the third one, \(R_{22}\), is set to 1 which multiplied by \(P.z\) leaves the value of \(P.z\) unchanged. We can conclude that the matrix generating a rotation of a point/vector around the z-axis has the following form:

$$
R_z(\theta)=
\begin{bmatrix}
 \cos(\theta) & -\sin(\theta) & 0\\
\sin(\theta) & \cos(\theta) & 0\\
0 & 0 & 1
\end{bmatrix}
$$

To find the matrices that could rotate a point around the x and y axis (or in the yz and xz planes), we can follow the same logic we used to find the matrix that rotates points and vectors around the z-axis (in the xy plane). We will leave this to you as an exercise but considering the information we have given for finding Rz, that should be pretty simple. If Rx is the matrix that generates a rotation around the x-axis and Ry is the matrix that produces a rotation around the y-axis, here is what these matrices look like:

$$
R_x(\theta)=
\begin{bmatrix} 
1 & 0 & 0 \\
0 & \cos(\theta) & -\sin(\theta) \\
0 & \sin(\theta) & \cos(\theta) \\
\end{bmatrix}
$$

$$
R_y(\theta)=
\begin{bmatrix}
\cos(\theta) & 0 & \sin(\theta) \\
0 & 1 &  0 \\
-\sin(\theta) & 0 & \cos(\theta) \\ 
\end{bmatrix}
$$

$$
R_z(\theta)=
\begin{bmatrix}
 \cos(\theta) & -\sin(\theta) & 0\\
\sin(\theta) & \cos(\theta) & 0\\
0 & 0 & 1 \\ \end{bmatrix}
$$

Remember that you multiply the point's coordinates by the coefficients contained in each **column** of these matrices to calculate the transformed point's x, y and z coordinates.

![Figure 5: rotations around the x- y- and z-axis. The arrow indicates the rotation direction for positive angles.](/images/geometry/rotation2.png?)

![Figure 6: if you use a left-hand coordinate system (left), wrap your fingers around the axis of rotation to find in which direction positive rotation values will rotate points and vectors. If you use a right-hand coordinate system, use the same procedure using your right hand instead.](/images/geometry/rothand.png?)

You can also use the mnemonic technique described in the chapter on coordinating the system to easily quickly determine which direction points or vectors will rotate if the rotation angle is positive. For example, for a right-hand coordinate system, wrap your fingers around the rotation axis (Figure 6). They will naturally indicate the direction in which positive rotation values rotate vectors and points (positive rotation is counter-clockwise). Repeat the procedure for a left-hand coordinate system but use your left hand instead (positive rotation is clockwise).

## Combining (Rotation) Matrices

In the previous chapter, we learned that multiplying matrices together combines their transformations. Now that we know how to rotate points around an individual axis, we can multiply Rx, Ry, and Rz together (using every possible combination) to create more complex rotations. If, for instance, you want to rotate a point around the x-axis and then the y-axis, we can create two matrices using the matrices Rx and Ry and combine them using matrix multiplication (Rx*Ry) to create a Rxy matrix encoding the two individual rotations:

$$R_{XY}=R_X*R_Y$$

Note that the order of rotation is essential and makes a difference. For example, if you rotate a point around the x-axis first and then the y-axis second, you will end up (in most cases) with a result different from a rotation around the y-axis and then around the x-axis. In most 3D packages such as Maya, 3DSMax, Softimage, Houdini, etc., it is possible to specify the order in which the rotations occur. For instance, the order can be xyz, ... (see in Maya the list of possible options).

## The Translation Matrix

We need to use [4x4] matrices to translate points using point-matrix multiplication. Therefore, this chapter is limited to [3x3] matrices; we will explain how translation works with matrices in [Transforming Points and Vectors](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-points-and-vectors).

## Rotation around an Arbitrary Axis

It is possible to write some code that will rotate a point or a vector around an arbitrary axis. However, writing a basic raytracer won't be necessary, and we will develop this topic in a future revision of this lesson after the other basic lessons are completed.
