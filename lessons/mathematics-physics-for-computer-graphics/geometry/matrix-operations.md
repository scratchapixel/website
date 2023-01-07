## Transpose

The transpose of a matrix \(M\) is another matrix that we write using the following convention: \(M^T\) (with the superscript T). We can describe the process of transposing a matrix in different ways. It can be seen as reflecting \(M\) over its main diagonal (from left to right, top to bottom) to obtain \(M^T\), writing the rows of \(M\) as the columns of \(M^T\) or reciprocally, writing the columns of \(M\) as the rows of \(M^T\). Computing the transpose of a matrix can be done with the following code:

```
Matrix44 transpose() const
{
    Matrix44 transpMat;
    for (uint8_t i = 0; i < 4; ++i) {
        for (uint8_t j = 0; j < 4; ++j) {
            transpMat[i][j] = m[j][i];
        }
    }
        
    return transpMat;
}
```

The idea is to swap the rows and columns, and since this operation can't be done in place, we need to assign the result to a new matrix returned by the function. Transposing matrices can be helpful when you want to convert matrices from a 3D application using row-major matrices to another using a column-major convention (and vice versa).

## Inverse

If multiplying point A by the Matrix M gives point B, multiplying point B by the inverse of the matrix M gives point A. In mathematics, a matrix inversion is usually written using the following notation:

$$M^{-1}$$

From this observation, we can write that:

$$MM^{-1}=I$$

Where I is the identity matrix, multiplying a matrix by its inverse gives the identity matrix.

<details>
In the chapter [How Does a Matrix Work](/lessons/mathematics-physics-for-computer-graphics/geometry/how-does-matrix-work-part-2), we have mentioned the case of the orthogonal matrix, which inverse can easily be obtained from computing its transpose. An orthogonal matrix is a square matrix with real entries whose columns and rows are orthogonal unit vectors. We will use this critical property to learn how to [transform normals](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-normals).
</details>

Matrix inversion is an essential process in 3D. We can use point- or vector-matrix multiplication to convert points and vectors. Still, moving the transformed points or vectors back into the coordinate system they were initially defined is sometimes helpful. For example, it is often necessary to transform the ray direction and origin in object space to test for a primitive-ray intersection. If there is an intersection, the resulting hit point is in object space and needs to be converted back into world space to be usable.

The lesson [Matrix Inverse](/lessons/mathematics-physics-for-computer-graphics/matrix-inverse/) will teach how to calculate the inverse of a matrix (only available in the old version of Scratchapixel for now). Developing even a basic renderer without being able to use matrices and their inverse would be limited, so we will be providing some code in this lesson for doing so. You can use this code without worrying too much about how it works and read this advanced lesson another time if you need more time to feel ready.

## Determinant of a Matrix

This part of the lesson will be written later.