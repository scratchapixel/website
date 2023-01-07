Rendering an image by keeping all the 3D objects and the camera at the origin would be limited. In essence, matrices play an essential role in moving objects, lights, and cameras around the scene so that you can compose your image the way you want.

## Introduction to Matrices: they Make Transformations Easy!

Matrices play an instrumental part in the graphics pipeline, and you will see them used regularly in the code of 3D applications. There is nothing complicated about matrices; if you fear them, it might be because you must fully comprehend what they represent and how they work (yet). So let's fix the situation.

In the previous chapter, we mentioned that it was possible to translate or rotate points using linear operators. For example, we showed that we could translate a point by adding some values to its coordinates. We also showed that it was possible to rotate a vector using trigonometric functions. Now, in short (and this is not a mathematical definition of what matrices are), a matrix combines all these transformations (scale, rotation, translation) into one single structure. Multiplying a point or a vector by this structure (the matrix) gives us a transformed point or vector. Combining these transformations means any combination of the following linear transformations: scale, rotation, and translation. For example, we can create a matrix that will rotate a point by 90 degrees around the x-axis, scale it by 2 along the z-axis (the scale applied to the point is (1, 1, 2)), and then translate it by (-2, 3, 1). We could do this by performing a succession of linear transformations on a point, but this would potentially mean writing a lot of code:

```
Vec3f translate(Vec3f P, Vec3f translateValue) { ... } 
Vec3f scale(Vec3f P, Vec3f scaleValue) { ... } 
Vec3f rotate(Vec3f P, Vec3f axis, float angle) { ... } 
... 
Vec3f P = Vec3f(1, 1, 1); 
Vec3f translateVal(-1, 2, 4); 
Vec3f scaleVal(1, 1, 2); 
Vec3f axis(1, 0, 0); 
float angle = 90; 
Vec3f Pt; 
Pt = translate(P, translateVal):  //translate P 
Pt = scale(Pt, scaleVal);  //then scale the result 
Pt = rotateValue(Pt, axis, angle);  //finally rotate the point 
```

This code could be more compact. But if we use a matrix, we can write:

```
Matrix4f M(...);  //set the matrix for translation, rotation, and scale. 
Vec3f P = Vec3f(1, 1, 1); 
Vec3f Ptranformed = P * M;  //do everything at once, translate, rotate, scale. 
```

Transforming P to achieve a similar effect is multiplying the point with a matrix (M). We are showing here what matrices are used for in the graphics pipeline and their advantages. In that particular example, we have told you that they can be used to combine any of the three fundamental geometric transformations we can perform on points and vectors (scale, translation, rotation) in a straightforward, fast, and compact way. What we have to do now is to explain to you how and why that works (it will take us a few chapters, though).

## Matrices, What Are They?

![Figure 1: a [4x4] matrix.](/images/geometry/rowcolumn.png?)

What are matrices? What secret do they hold? Instead of answering with an abstract mathematical definition, we will start with real matrix examples. Once we have seen a couple more concrete examples extending the concept to its generic/mathematical form will be easier. If you have read a few CG books already, you may have seen matrices mentioned in quite a few places, and they often appear as a two-dimensional array of numbers. To define a two-dimensional array of numbers, we use the standard notation **m x n**, where m and n are two numbers that represent the size of this array. As you may have guessed, m and n respectively represent the **number of rows and columns** of the matrix. Rows are the horizontal lines of numbers in the 2D array, and columns are the vertical ones. Here is an example of a [3x5] matrix:

$$
\begin{bmatrix}
1&3&7&9&0\\
3&3&0&8&3\\
9&1&0&0&1
\end{bmatrix}
$$

We will denote the matrix numbers and the matrix coefficients (you might come across the term entry or element, but coefficient is often used in CG), and we usually use the subscripts i, and j to point to a particular coefficient in the matrix. Matrices are generally written with capital letters (M, A, B, etc.).

\(M_{ij}\) where \(i\) refers to the row number and \(j\) refers to the column number.

We will make a lot of simplifications on matrices for now. One of them is that in CGI, we primarily use squared matrices. These are matrices whose numbers m and n are equal. Typically, in CG, we will be interested in 3x3 or 4x4 matrices, and we will tell you in the following chapter what they are and how to use them. These matrices are generally called **square matrices** (the matrix [m x n] is a square matrix if m = n). This is a simplification because m and n can take any value and don't have to be equal. For example, you can create a 3x1 matrix, a 6x6 matrix, or a 4x2 matrix. They are all valid matrices. But as we said, we will mainly use 3x3 and 4x4 matrices in CG.

A [3x3] \(\begin{bmatrix} 7&4&3\\ 2&0&3\\ 3&9&1\\ \end{bmatrix}\) and [4x4] \(\begin{bmatrix} 7&1&4&3\\ 2&0&0&3\\ 3&1&9&1\\ 6&6&5&4\\ \end{bmatrix}\) matrix.

Here is an example of how we can implement a 4x4 matrix class in C++ (note that we use the template mechanism in case we need the matrix to use a float or double precision):

```
template<typename T> 
class Matrix44 
{ 
public: 
    Matrix44() {} 
    const T* operator [] (uint8_t i) const { return m[i]; } 
    T* operator [] (uint8_t i) { return m[i]; } 
    // initialize the coefficients of the matrix with the coefficients of the identity matrix
    T m[4][4] = {{1,0,0,0},{0,1,0,0},{0,0,1,0},{0,0,0,1}}; 
}; 
 
typedef Matrix44<float> Matrix44f; 
```

<details>
These operators in the Matrix44 class:

```
const T* operator [] (uint8_t i) const { return m[i]; } 
T* operator [] (uint8_t i) { return m[i]; } 
```

They are sometimes called access operators or accessors. They are used to access the matrix coefficients without explicitly accessing the member variable m[4][4]. Typically, you would access the coefficients that way:

```
Matrix44f mat; 
mat.m[0][3] = 1.f; 
```

But with the access operators, you can write:

```
Matrix44f mat; 
mat[0][3] = 1.f; 
```
</details>

## Matrix Multiplication

Matrices can be multiplied with each other, and this operation is at the heart of the point- or vector-matrix transformation process. The result of matrix multiplication (the technical term is **matrix product**, the product of two matrices) is another matrix:

$$M_3=M_1 * M_2$$

![Figure 2: a matrix to transform A to C can be obtained by multiplying a matrix M1 that transform A to B with a matrix M2 that transform point B to C. The multiplication of any matrix combination that transforms in successive steps A to C will give matrix M3.](/images/geometry/matrixmult.png?)

Suppose you remember what we briefly mentioned in the introduction. In that case, a matrix concisely defines a combination of linear transformations that can be applied to points and vectors (scale, rotation, translation). We have yet to explain how that works, but we will address it soon; what's important to understand now is that matrix multiplication combines the effect of two other matrices in one matrix. In other words, the transformation that each matrix M1 and M2 would operate on a point or a vector can be combined in one single matrix M3\. For example, imagine you need to transform a point from A to B using matrix M1 and then transform B to C using matrix M2\. Multiplying M1 by M2 gives a matrix M3 which directly transforms A to C. A matrix obtained by multiplying two matrices is not different from the other two. What's important to note here is that if you have two other matrices, M4 and M5, that transform A to D and D to C, then the multiplication of M4 with M5 will give you M3 again (there is a unique matrix for each particular transformation).

Now there is a rule about matrix multiplication which is optional to know if you deal with 4x4 matrices (and you will understand why soon). Still, for your general knowledge of the subject, we will explain it here (it will become essential to remember when we will deal with point-matrix and vector-matrix multiplication). Two matrices, M1 and M2, can only be multiplied if the number of columns in M1 equals the number of rows in M2. In other words, if two matrices can be written as m x p and p x n, they can be multiplied, giving a matrix of size m x n. Two matrices, \(p x m\) and \(n x p\) can not be multiplied because m and n are unequal. A 4x2 and 2x3 matrices can be multiplied, giving a 4x3 matrix. The multiplication of two 4x4 matrices gives a 4x4 matrix (this rule isn't so essential for us because we will almost always use a 4x4 matrix, so we generally won't care about whether matrices can be multiplied or not).

$$[M \times P] * [P \times N] = [M \times N]$$

Let's see how we multiply two matrices together, a mathematical operation on the coefficients of the two input matrices. In other words, we are interested in how we compute the coefficients of the new matrix. It is pretty simple as long as you remember the rule. We said previously that the coefficients in a matrix were defined by their row and column indices. Notation-wise, we use the subscripts i and j to denote these row and column indices. So imagine that we want to find the value of the coefficient Mi,j in the matrix M3. Let's say that i=1 and j=2 (note that index 0 indicates either the first row or the first column of the matrix. Index 3 indicates the last row or column. **Arrays start at index 0** in C++). So to compute M3(1,2), we select all the coefficients of the second row in M1 (where M1 is a 4x4 matrix) and all the coefficients of the third column in M2 (where M2 is also a 4x4 matrix). That gives us two sequences of four numbers that we will multiply with each other and sum up in the following way:

$$
M1=
\begin{bmatrix}
c_{00}&c_{01}&c_{02}&c_{03}\\
\color{red}{c_{10}}&\color{red}{c_{11}}&\color{red}{c_{12}}&\color{red}{c_{13}}\\
c_{20}&c_{21}&c_{22}&c_{23}\\
c_{30}&c_{31}&c_{32}&c_{33}\\
\end{bmatrix} \text{ } M2=
\begin{bmatrix}
c_{00}&c_{01}&\color{red}{c_{02}}&c_{03}\\
c_{10}&c_{11}&\color{red}{c_{12}}&c_{13}\\
c_{20}&c_{21}&\color{red}{c_{22}}&c_{23}\\
c_{30}&c_{31}&\color{red}{c_{32}}&c_{33}\\
\end{bmatrix}
$$

$$M3_{12}=
\begin{array}{l}
    M1_{10}*M2_{02} + \\
    M1_{11}*M2_{12} + \\
    M1_{12}*M2_{22} + \\ 
    M1_{13}*M2_{32}
\end{array}
$$

We can use this process for all the coefficients of M3: use the row and column index of the coefficient we want to calculate and use these indices to select the coefficients of the corresponding row in M1 (M1(i,0), M1(i,1), M1(i,2), M1(i,3)) and select the coefficients for the corresponding column in M2 (M2(0,j), M2(1,j), M2(2,j), M3(3,j). Once we have these numbers, we combine them using the above formula. Multiply all the coefficients of the same index with each other and sum up the results:

$$
M3_{ij}=
\begin{array}{l}
    M1_{i0}*M2_{0j} + \\
    M1_{i1}*M2_{1j} + \\
    M1_{i2}*M2_{2j} + \\ 
    M1_{i3}*M2_{3j}
\end{array}
$$

Let's see how we could code this operation in C++. First, let's define a matrix as a two-dimensional array of 4-by-4 floats. Then, here is the function that can be used to multiply two matrices together:

```
Matrix44 operator * (const Matrix44& rhs) const
{
    Matrix44 mult;
    for (uint8_t i = 0; i < 4; ++i) {
        for (uint8_t j = 0; j < 4; ++j) {
            mult[i][j] = m[i][0] * rhs[0][j] +
                         m[i][1] * rhs[1][j] +
                         m[i][2] * rhs[2][j] +
                         m[i][3] * rhs[3][j];
        }
    }

    return mult;
}
```

When you know how the multiplication of two matrices is obtained, it is not hard to observe that the multiplication of M1 by M2 gives a different result than the multiplication of M2 by M1. Matrix multiplication, indeed, is not **commutative**. So M1* M2 gives a different result than M2*M1.

## Summary

We have yet to explain how and why matrices work but do not worry. All these essential things will be presented in the next chapter. From this chapter, you must remember that a matrix is a two-dimensional array of numbers. The size of the matrix is denoted \(m x n\), where m is the number of rows and n is the number of columns. You have learned that matrices can be multiplied only if the matrix on the left side of the multiplication has a number of columns equal to the number of rows on the right inside of the multiplication. For instance, two matrices of \(m x p\) and \(p x n\) can be multiplied by each other. The resulting matrix combines the transformation of the two matrices used in multiplication. For example, if M1 transforms a point from A to B and M2 transforms a point from B to C, then if M3 is the result of M1 multiplied by M2, M3 will transform this point from A to C. Finally, we have learned how to compute a matrix's coefficients resulting from matrix multiplication. It is also important to remember that matrix multiplication is not commutative. We will need to pay attention to the order in which we multiply matrices with each other. This order matters; if your code doesn't work, you should check the order in which matrices are multiplied by each other.