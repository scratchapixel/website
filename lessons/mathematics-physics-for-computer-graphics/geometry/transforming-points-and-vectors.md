## Transforming Points

We have introduced almost all we need to know to write the code that will transform points using matrices. However, even though translation is the easiest linear operator that can be applied to a point, we have yet to mention it often in the previous chapter. To get the translation working with the theory of matrix multiplication, we need to change the point structure, which might need to be clarified.

As we mentioned in the last two chapters, a matrix-matrix multiplication can only work if the two matrices involved have a compatible size. That is, if they have the size m x p and p x n. Let's keep that in mind. Let's start with a 3x3 identity matrix. We know that a point multiplied by this matrix has its coordinates unchanged. Let's see what changes we need to bring to that matrix to handle translation. Translation on a point is nothing more than adding a number to each of its coordinates (these numbers can be positive or negative). For instance, if we want to move the point (1, 1, 1) to the coordinate (2, 3, 4), we need to add the values 1, 2, and 3, respectively, to each of the point's x, y, and z coordinates. It is straightforward. Note that from now on, we will keep looking at points and vectors as a matrix of size 1x3.

$$
\begin{array}{l}
P'.x = P.x + Tx\\
P'.y = P.y + Ty\\
P'.z = P.z + Tz
\end{array}
$$

Now let's get back to the code that transforms a point using a matrix:

$$
\begin{array}{l}
P'.x = P.x * M_{00} + P.y * M_{10} + P.z * M_{20}\\
P'.y = P.x * M_{01} + P.y * M_{11} + P.z * M_{21}\\
P'.z = P.x * M_{02} + P.y * M_{12} + P.z * M_{22}
\end{array}
$$

What do we need to get the rotation matrix extended to handle translation as well? We need to have a fourth term to the right to encode the translation. Something like this:

$$
\begin{array}{l}
P'.x = P.x * M_{00} + P.y * M_{10} + P.z * M_{20} + T_X\\
P'.y = P.x * M_{01} + P.y * M_{11} + P.z * M_{21} + T_Y\\
P'.z = P.x * M_{02} + P.y * M_{12} + P.z * M_{22} + T_Z
\end{array}
$$

Remember that we want to develop a matrix that encodes scale, rotation, and translation. So, we need to get Tx, Ty, and Tz to fit within the code of the point-matrix multiplication (and store these thee values somewhere in the matrix). Look at the first line for now. Note that to calculate x', we only use the coefficients of the matrix's first column. So, if the column had four coefficients instead of three, Tx would be \(M_{30}\). The same reasoning can be done with Ty and Tz. We would then get the following:

$$
\begin{array}{l}
P'.x = P.x * M_{00} + P.y * M_{10} + P.z * M_{20} + M_{30}\\
P'.y = P.x * M_{01} + P.y * M_{11} + P.z * M_{21} + M_{31}\\
P'.z = P.x * M_{02} + P.y * M_{12} + P.z * M_{22} + M_{32}
\end{array}
$$

However, this assumes that our matrix now has the size 4x3 and not 3x3. This is alright. We said that matrices could have any size. However, we know that matrix multiplication can only be valid if their sizes are compatible. But we now try to multiply a point as a 1x3 matrix with a 4x3 matrix, and theory tells us that this is impossible. What shall we do? The solution is simple. We will add one additional column to the point to turn it into a 1x4 matrix and set the fourth coefficient of this point to 1. Our point now looks like this (x, y, z, 1). In computer graphics, it is called a **homogeneous point** (or a point with **homogeneous coordinates**). With such a point, we can easily encode translation in our matrix. See how it magically falls into place in the following code:

$$
\begin{array}{l}
P'.x = P.x * M_{00} + P.y * M_{10} + P.z * M_{20} + 1 * M_{30}\\
P'.y = P.x * M_{01} + P.y * M_{11} + P.z * M_{21} + 1 * M_{31}\\
P'.z = P.x * M_{02} + P.y * M_{12} + P.z * M_{22} + 1 * M_{32}
\end{array}
$$

This is the theory. To encode translation, scale, and rotation in a matrix, we need to deal with points that have homogeneous coordinates. But because the fourth value is always 1, we don't explicitly define it in the code. Instead, we only define x, y, and z and assume that there is a fourth value. The point-matrix code now looks like this:

$$
\begin{array}{l}
P'.x = P.x * M_{00} + P.y * M_{10} + P.z * M_{20} + M_{30}\\
P'.y = P.x * M_{01} + P.y * M_{11} + P.z * M_{21} + M_{31}\\
P'.z = P.x * M_{02} + P.y * M_{12} + P.z * M_{22} + M_{32}
\end{array}
$$

Our matrix is now a 4x3 matrix. Let's go from a 4x3 matrix to our final 4x4 matrix, the CG's most commonly used form. The fourth columns play a role in perspective projection and for some other types of transformations that are not very common (such as the shear transformation), but generally, it is set to (0, 0, 0, 1). What happens when the coefficient of this column has different values than the default (we said it's uncommon but happens sometimes)? Before answering this question, we first need to learn a few more things about homogenous points.

## The Trick About Homogeneous Points

Presenting a point as a homogeneous point is necessary to allow point multiplication by [4x4] matrices; however, in the code, this is only done implicitly since, as we have explained, w is always 1. Therefore, our Point C++ class will not define the point type with four floats but only with three (x, y and z). Technically, if we were to make a multiplication of a homogeneous point by a [4x4] matrix, the w coordinate of the transformed point would be obtained by multiplying the point's coordinates by the coefficients of the matrix's fourth column. However, as we mentioned, this column is almost always set to (0, 0, 0, 1). In that case, the value of w' (the w coordinate of the transformed point) should be 1 (w'=x*0+y*0+z*0+w(=1)*1=1) and the resulting transformed x', y' and z' coordinates can be used directly. But as we also mentioned briefly, this fourth column is not always set to (0, 0, 0, 1), mainly when you deal with projection matrices (matrices that project points to the screen). In these exceptional cases, the result for w' can be different than 1 (which is intentional). Still, for this point to be usable as a Cartesian point, we need to normalize w' back to 1 by dividing it by itself, which requires dividing the other coordinates (x', y', and z') by w' as well. In pseudo-code, it gives something like that:

```
P'.x = P.x * M00 + P.y * M10 + P.z * M20 + M30;
P'.y = P.x * M01 + P.y * M11 + P.z * M21 + M31;
P'.z = P.x * M02 + P.y * M12 + P.z * M22 + M32;
w'   = P.x * M03 + P.y * M13 + P.z * M23 + M33;
if (w' != 1 && w' != 0) {
    P'.x /= w', P'.y /= w', P'.z /= w';
}
```

As you can see, we don't need to declare a w coordinate in the Point's type. We can compute a value for w' on the fly as we assume implicitly that the point we are transforming is a Cartesian point which you can see as a homogeneous point whose w coordinate is not declared explicitly (because it's always equal to 1). However, if the matrix we are multiplying the point with is a projection matrix, the result of w' might be different than 1. In this particular case, we need to normalize all the coordinates of P' to set it back to 1. Once this is done, we get the point we can use in our Cartesian coordinate system again.

All you need to remember is that you will generally never have to care about homogeneous coordinates except when points are multiplied by a perspective projection matrix. However, you will probably not encounter this issue if you work on a ray tracer, as this particular type of matrix is not used in ray tracing. If you still need help understanding what this w coordinate is and what it is used for, check the [Perspective and Orthographic Projection Matrix](/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/) lesson in the 3D Basic Rendering section. You will learn how to project 3D points onto the image plane using perspective projection. The concept of a homogeneous point will then make more sense.

When it comes to implementing this function in C++, it can be dealt with in two ways:

- Some developers like the code for point-matrix multiplication to always compute a value for w' and divide the coordinates of the transformed points by the value of w' if it is different than 1. However, this is only useful when we multiply points by projection matrices which is rare (particularly in raytracers). In 99% of cases, computing w' and checking if it is different than 1 wastes CPU time. 

- One might ignore w and w' and assume the point-matrix multiplication code will be used with matrices whose fourth column is always set to (0, 0, 0, 1). When dealing with the particular case of projection matrices, you can always create a second function that will compute w' and divide x' y' and z' by w's value. 

You can choose between a generic but not completely optimized and a more optimized solution that requires two functions instead of one. For the sake of clarity, let's go with the generic option:

```
void multVecMatrix(const Vec3&lt;T&gt; &src, Vec3&lt;T&gt; &dst) const
{
    dst.x = src.x * m[0][0] + src.y * m[1][0] + src.z * m[2][0] + m[3][0];
    dst.y = src.x * m[0][1] + src.y * m[1][1] + src.z * m[2][1] + m[3][1];
    dst.z = src.x * m[0][2] + src.y * m[1][2] + src.z * m[2][2] + m[3][2];
    T w = src.x * m[0][3] + src.y * m[1][3] + src.z * m[2][3] + m[3][3];
    if (w != 1 && w != 0) {
        dst.x = x / w;
        dst.y = y / w;
        dst.z = z / w;
    }
}
```

## Transforming Vectors

Vectors are more straightforward to transform than points. Vectors, as we said in the preamble of this lesson, represent direction, whereas points represent the position in space. As such, **vectors do not need to be translated** because their position is, in fact, meaningless. With vectors, we are only interested in the direction they point and, eventually, their length, which is sometimes the information we need to solve geometric or shading problems. Vectors can be transformed like we transformed points, but we can remove the part of the code responsible for the translation bit. The code used to transform vectors looks like this (compare it with the code to transform points).

```
V'.x = V.x * M00 + V.y * M10 + V.z * M20;
V'.y = V.x * M01 + V.y * M11 + V.z * M21;
V'.z = V.x * M02 + V.y * M12 + V.z * M22;
```

Here is the code that transforms vectors:

```
void multDirMatrix(const Vec3&lt;T&gt; &src, Vec3&lt;T&gt; &dst) const
{
    dst.x = src.x * m[0][0] + src.y * m[1][0] + src.z * m[2][0];
    dst.y = src.x * m[0][1] + src.y * m[1][1] + src.z * m[2][1];
    dst.z = src.x * m[0][2] + src.y * m[1][2] + src.z * m[2][2];
}
```

## Transforming Normals

As strange as it sounds, normals are like vectors and can be transformed using the same code. However, it is more complex, and we will explain why in the chapter on [Transforming Normals](#).

## Conclusion

This chapter teaches us why we use [4x4] rather than [3x3] matrices. The coefficients \(c_{30}\) \(c_{31}\) and \(c_{32}\) hold the translation values. Now that the matrix has size [4x4], we need to extend the point size by adding an extra coordinate. We can do this by implicitly treating points as Homogenous points but to continue using them in a Cartesian coordinate system (as Cartesian points), we need to be sure that w, this fourth coordinate, is always set to 1. Most of the time, the matrices we use to transform a point will have their fourth column set to (0, 0, 0, 1), and with these matrices, the value of w' should always be 1. However, in special cases (projection matrix, shear transform), the value of w' might be different than 1, in which case you will need to normalize it (we divided w' by itself), which requires to also divide the other transformed coordinates x', y' and z' by w'.

<details>
Matrices are not the only method to "encode" or store transformations. You can also, for instance, represent a rotation using a method proposed initially by Euler. The idea is to define a rotation, in this case, as a vector and an angle representing a rotation around that vector. You can also use a technique developed by [Benjamin Olinde Rodrigues](https://en.wikipedia.org/wiki/Olinde_Rodrigues). Given an axis \(\hat r\), an angle \(\theta\) and a point \(p\), the rotation is given by the following equation: 

$$
R(\hat r, \theta, p) = p \cos \theta + (\hat r \times p) \sin \theta + \hat r( \hat r \cdot p)(1 - \cos \theta)
$$

While uncommon, both techniques are used to solve problems in computer graphics from time to time. Rotations in computer graphics are also commonly done using **quaternions**. 

Matrices have certain limitations. For example, a problem called [gimbal lock](https://en.wikipedia.org/wiki/Gimbal_lock) occasionally occurs with matrices. A gimbal lock induces a discontinuous jump in one or more axes' orientations.

Using matrices for interpolating objects' rotations can also be a problem when simulating objects' motion blur, for which interpolating objects' transformations is required.

For these reasons, quaternions are generally preferred though they are harder to understand.
</details>