In this lesson, we will show how the inverse of a matrix can be derived using a technique known as the Gauss-Jordan (or reduced row) elimination method. Finding the inverse of a matrix implies a couple of things, starting with the fact that the matrix is invertible in the first place (a matrix is not necessarily invertible). First, to be invertible, a matrix has to be a **square matrix** (it has as many rows as it has columns, for instance, 2x2, 3x3, 4x4, etc.), and also the determinant of the matrix has to be different than zero (to learn about the determinant of a matrix refer to [Geometry: Matrix Operations](/lessons/mathematics-physics-for-computer-graphics/geometry/matrix-operations)). Any matrix with a zero determinant is said to be singular (meaning it is not invertible).

Before we get started, remember that a matrix can be seen as a system of linear equations that can be solved using what we call [**row elementary operations**](https://en.wikipedia.org/wiki/Elementary_matrix#Operations). Row elementary operations have the property to preserve the solution set by the matrix. There are three such operations: 

- **Operation 1**: swapping rows of a matrix.
- **Operation 2**: multiplying each coefficient of a row by a non-zero constant \(k\). Note that we say multiplying here, which implies that this can be a division too. We can multiply coefficients by \(\frac{1}{2}\), which is the same as dividing the coefficients by 2. Similarly, this constant can be positive or negative, 2 or -2.
- **Operation 3**: replacing a row with the sum of itself and the multiple of another row. For instance, we can add the coefficients of row 4 to those of row 1. But we can also, in the process, multiply the coefficients of row 4 by some constant \(k\) as follows:

$$
\begin{array}{l}
M_{11} += M_{41} * k,\\
M_{12} += M_{42} * k,\\
M_{13} += M_{43} * k,\\
M_{14} += M_{44} * k
\end{array}
$$

These are illustrated with the following examples (row switching, row multiplication, and row addition):

![](/images/matrix-inverse/mi-rowop.png)

In the second example, we multiply all the coefficients of row 1 by 1/2 (or divide them by 2) and the coefficients of row 3 by 2. Finally, in the third example, we add the coefficients from row 1 to the coefficients of row 2. 

The idea behind the Gauss-Jordan elimination is to use these elementary row operations to transform the 4x4 matrix on the left inside of the augmented matrix into the identity matrix (we say that M is row-reduced) as shown below. We say that we [augment M by identity](https://en.wikipedia.org/wiki/Augmented_matrix). We obtain the inverse matrix by performing the same row operations to the 4x4 identity matrix on the right inside of the augmented matrix.

The Gauss-Jordan elimination method works as follows:

- We will start with two matrices. We want to invert the matrix \(M\) alongside another matrix that we will set at the start of the process to the identity matrix. An identity matrix is a square matrix with ones on the diagonal and zeros elsewhere (the Matrix class constructor sets the matrix to the identity matrix by default). Our matrix \(M\) is augmented. 

$$
[MI]=
\begin{bmatrix}
M_{11} & M_{12} & M_{13} & M_{14}\\
M_{21} & M_{22} & M_{23} & M_{24}\\
M_{31} & M_{32} & M_{33} & M_{34}\\
M_{41} & M_{42} & M_{43} & M_{44}
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & 0\\
0 & 1 & 0 & 0\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{bmatrix}
$$

- We then have to use row elementary operations to transform the original \(M\) matrix into the identity matrix. This process is called **row reduction**. While we do so, we apply the same operations to the second matrix in parallel. 

- Once the process is finished (if we have been successful in reducing \(M\) to the identity matrix), the second matrix will end up being \(M^{-1}\) our inverted matrix. At this point, you can decide to either return \(M^{-1}\) (the method named `inverse` below) or copy the inverted matrix into \(M\) (the method called `invertIt` below).

$$
[IM^{-1}]=
\begin{bmatrix}
1 & 0 & 0 & 0\\
0 & 1 & 0 & 0\\
0 & 0 & 1 & 0\\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
M'_{11} & M'_{12} & M'_{13} & M'_{14}\\
M'_{21} & M'_{22} & M'_{23} & M'_{24}\\
M'_{31} & M'_{32} & M'_{33} & M'_{34}\\
M'_{41} & M'_{42} & M'_{43} & M'_{44}
\end{bmatrix}
$$

```
Matrix44& Matrix44::invertIt()
{
    Matrix44 inv; // start from identity matrix
    ...
    *this = inv;
    return *this;
}

Matrix44 Matrix44::inverse(Matrix44& M) const
{ Matrix44 temp(*this); temp.invertIt(); return temp; }

```

Let's see how this is done.

## Step 1: Set The Row So That The Pivot Is Different Than Zero

The coefficients making the diagonal of the matrix are called the **pivots** of the matrix.

![](/images/matrix-inverse/mi-pivots.png?)

As we mentioned earlier, the goal of the matrix inversion process is to use the row elementary operations to set the pivot of each column to 1 and all the other coefficients to 0 (at the end of this process, we will get the identity matrix). To achieve this, the best is to row-reduce each column one after the other starting from the left. Proceeding from left to right guarantees that as we move from column to column, we do not change the values of the columns which have already been row-reduced.

![](/images/matrix-inverse/mi-columns.gif)

We will first examine the value of the pivot coefficient for the current column (the column being processed). This value has to be different than zero. If it is other than zero, we can directly move to step two. If it is equal to zero, we will need to find another row in the matrix for this column whose value is different than zero and swap these two rows (the first of the permitted row elementary operations). However, more than one row may have values other than zero. So which one should we pick in this case? We will select the row whose absolute value is the highest.

![](/images/matrix-inverse/mi-swap.gif)

In the example on the right, we are looking at the pivot coefficient of the second column. Because the value of that coefficient is zero, we swap row 2 with the row of the second column whose absolute value is the greatest (in this example, row 3).

If we can't find another value for the pivot coefficient (that is, if all the other values in the column are zero), then the matrix can't be inverted and is singular (line 11).

If the current column we are processing has a valid pivot coefficient, we can proceed to the next step.

```
Matrix<T, N>& invertIt() 
{ 
    Matrix<T, N> inv; 
    for (unsigned column = 0; column < N; ++column) { 
        // Swap row in case our pivot point is not working
        if (m[column][column] == 0) { 
            size_t big = column; 
            for (unsigned row = 0; row < N; ++row) 
                if (fabs(m[row][column]) > fabs(m[big][column])) big = row; 
            // Print this as a singular matrix, return identity?
            if (big == column) fprintf(stderr, "Singular matrix\n"); 
            // Swap rows                               
            else for (unsigned j = 0; j < N; ++j) { 
                std::swap(m[column][j], m[big][j]); 
                std::swap(inv[column][j], inv[big][j]); 
            } 
        } 
        ... 
    } 
} 
```

## Step 2: Set Each Coefficient In The Column To 0

The Gauss-Jordan elimination method at this step processes the column one by one from left to right. We will row-reduce each coefficient of each column apart from the pivot coefficients, which we won't touch for now. In order, \(M_{21}\) (we skip \(M_{11}\) since it's a pivot coefficient), \(M_{31}\), \(M_{41}\), then \(M_{12}\) (we skip \(M_{22}\) since it's a pivot coefficient), \(M_{32}\), \(M_{42}\), etc. until \(M_{34}\) (we skip \(M_{44}\) since it's a pivot coefficient).

$$
\begin{bmatrix}
pivot  & M_{12} & M_{13} & M_{14}\\
M_{21} & pivot  & M_{23} & M_{24}\\
M_{31} & M_{32} & pivot  & M_{34}\\
M_{41} & M_{42} & M_{34} & pivot
\end{bmatrix}
$$

Now, remember that we can only use **row elementary operations**. That means you can only modify one single coefficient through a series of operations on that coefficient by changing all the other coefficients from the same row with the same set of operations. If you don't do so, you do not preserve the solution set by the matrix. That's the condition by which **row-reduction** works. Do whatever you want to your coefficient, as long you process all the coefficients in the row that this coefficient is on with the same set of row elementary operations. You can multiply all the coefficients in a row by the same constant and swap rows, and add the coefficients of a given row to those of another row.

Let's see how this works in practice. Let's imagine we are about to process the second column. Let's begin with the first coefficient \(\textcolor{red}{M_{12}}\). Let's call it @@\rJoe@@. Let's imagine that @@\rJoe@@'s value is 2. Now, remember that the goal is to somehow make @@\rJoe@@ 0 using one or a combination of the three possible row elementary operations. To make @@\rJoe@@ 0, we should subtract the value 2. That's pretty elementary, but how? Imagine that the coefficient \(\textcolor{blue}{M_{22}}\), the coefficient right below @@\rJoe@@, is equal to 4. Note that coincidentally, \(\textcolor{blue}{M_{22}}\) is also @@\rJoe@@'s column pivote coefficient. @@\rJoe@@ is in column 2, and the pivot coefficient of column 2 is \(\textcolor{blue}{M_{22}}\). 

$$
\begin{bmatrix}
M_{11} & \textcolor{red}{M_{12}} & M_{13} & M_{14}\\
M_{21} & \textcolor{blue}{M_{22}} & M_{23} & M_{24}\\
M_{31} & M_{32} & M_{33} & M_{34}\\
M_{41} & M_{42} & M_{34} & M_{44}
\end{bmatrix}
\;=\;
\begin{bmatrix}
x & \textcolor{red}{2} & x & x\\
x & \textcolor{blue}{4} & x & x\\
x & x & x & x\\
x & x & x & x
\end{bmatrix}
\;=\;
\begin{bmatrix}
x & \textcolor{red}{Joe} & x & x\\
x & \textcolor{blue}{pivot} & x & x\\
x & x & x & x\\
x & x & x & x
\end{bmatrix}
$$

Operation 3 says that we can add coefficients of any of the matrix rows to our current row coefficients. If we use the coefficient of the row that is just below @@\rJoe@@, \(\textcolor{blue}{M_{22}}\) (which we know is equal to 4), we get 6:

$$2 + 4 = 6$$

That's different from what we are looking for. However, remember that operation 2 says that you can also multiply the coefficients of a row by a constant. Let's imagine that this constant for now is -1. If we multiply \(\textcolor{blue}{M_{22}}\) by this constant, we get:

$$2 + -1 * 4 = -2$$

Not yet 0, but we are getting a step closer. What if our constant was equal to \(-\frac{1}{2}\) or \(-\frac{2}{4}\)

$$2 - \frac{2}{4}4 = 0$$

Now we canceled out the value of coefficient \(\textcolor{red}{M_{12}}\) and made it 0. Maybe you have noticed that 2 is the value of the coefficient \(\textcolor{red}{M_{12}}\) itself (Joe) and 4 is the value of the coefficient \(\textcolor{blue}{M_{22}}\), which happens to be the pivot coefficient of the second column, the column than Joe belongs to.

Keep in mind that the operations we applied to Joe also need to be applied to all the coefficients of the row Joe belongs to. That's the rule if you are to use row-elementary operations. So every coefficient on the row that the coefficient being canceled out belongs to needs to receive the same treatment as the one given to that coefficient (Joe). Like so for our Joe example:

$$
\begin{array}{l}
M_{11} &-=& k * M_{21},\\
\textcolor{red}{M_{12}} &-=& k * M_{22},\\
M_{13} &-=& k * M_{23},\\
M_{14} &-=& k * M_{24}.
\end{array}
$$

With:

$$k = -\frac{\textcolor{red}{M_{12}}}{\textcolor{blue}{M_{22}}}$$

Let's now generalize:

!!!
- For every coefficient in \(M_{ij}\) in the matrix where \(i\) is the coefficient's row and \(j\) its column that is not a pivot coefficient (proceeding in order through the columns first, and then for each column from top to bottom).
- Create a constant \(k\) that's made up of \(M_{ij}\) divided by the pivot coefficient of the column \(i\): \(M_{ii}\). If you process \(M_{12}\) (row 1, column 2), then use the pivot coefficient \(M_{22}\) (column 2). If you process \(M_{14}\) (row 1, column 4), then use the pivot coefficient \(M_{44}\) (column 4). Etc.
- Negate that number. 
- Multiply the coefficients of row **\(j\)** (note that \(j\) is the coefficient \(M_{ij}\)'s column) by the constant \(k\). That's allowed by operation 2. We always pick row \(j\) for this operation where again \(j\) is \(M_{ij}\)'s column. If you process \(M_{12}\) (row 1, column 2), then use the coefficients from row 2. If you process \(M_{14}\) (row 1, column 4), then use the coefficients from row 4. Etc.
- And finally, add the weighted coefficients from row \(j\) to the coefficients of row \(i\) (\(M_{ij}\)'s row). That's allowed by operation 3.
- \(M_{ij}\) is now equal to 0.
!!!

Keep in mind that we loop through the columns first (we loop through \(j\)) and that for each column, we process the coefficients from top to bottom (for each given \(j\), we loop through \(i\)), and skip the column's pivot coefficient (when \(i == j\)).

And the end of this process, all of these coefficients should be equal to 0 (except the pivot coefficients, of course).

In C++ code, we get the following:

```
// go left to right
for (unsigned column = 0; column < N; ++column) {
...
    // Make each row in column = 0. Proceed top to bottom
    for (unsigned row = 0; row < N; ++row) { 
         // skip pivot coefficients
		if (row != column) {
            // that's our constant k
            T coeff = m[row][column] / m[column][column]; 
            if (coeff != 0) { 
                for (unsigned j = 0; j < N; ++j) {
                    m[row][j] -= coeff * m[column][j]; 
                    inv[row][j] -=  coeff * inv[column][j]; 
                } 
                // Set the element to 0 for safety
                m[row][column] = 0; 
            } 
        } 
    }
}
```

Note how we apply the same operations to the right-hand side of the augmented matrix.

## Step 3: Scale All The Pivots Coefficients To 1

At the end of step 2, all the matrix coefficients should be 0 except the pivots coefficients which we need to set to 1. To do so, we need to scale the coefficients by their value. Similarly to what we have done in step 2, we will loop over each row, then each column, and divide the current coefficient by the pivot coefficient value of the current row:

```
for (unsigned row = 0; row < N; ++row) { 
    for (unsigned column = 0; column < N; ++column) { 
        mat.m[row][column] /= m[row][row]; 
    } 
} 
```

Finally, we set the result of the current matrix with the result of our right inside matrix (`inv` in the code). We have inverted the matrix using the Gauss-Jordan elimination technique.

Here is the complete code of the method:

```
Matrix<T, N>& invertIt() 
{ 
    Matrix<T, N> inv; 
    for (unsigned column = 0; column < N; ++column) { 
        // Swap row in case our pivot point is not working
        if (m[column][column] == 0) { 
            unsigned big = column; 
            for (unsigned row = 0; row < N; ++row) 
                if (fabs(m[row][column]) > fabs(m[big][column])) big = row; 
            // Print this as a singular matrix, return identity?
            if (big == column) fprintf(stderr, "Singular matrix\n"); 
            // Swap rows                               
            else for (unsigned j = 0; j < N; ++j) { 
                std::swap(m[column][j], m[big][j]); 
                std::swap(inv[column][j], inv[big][j]); 
            } 
        } 
        // Make each row in the column equal to 0  
        for (unsigned row = 0; row < N; ++row) { 
            if (row != column) { 
                T coeff = m[row][column] / m[column][column]; 
                if (coeff != 0) { 
                    for (unsigned j = 0; j < N; ++j) { 
                        m[row][j] -= coeff * m[column][j]; 
                        inv[row][j] -= coeff * inv[column][j]; 
                    } 
                    // Set the element to 0 for safety
                    m[row][column] = 0; 
                } 
            } 
        } 
    } 
    // Set each element of the diagonal to 1
    for (unsigned row = 0; row < N; ++row) { 
        for (unsigned column = 0; column < N; ++column) { 
            inv[row][column] /= m[row][row]; 
        } 
    } 
    *this = inv;

    return *this; 
} 
```

## What's Next?

There are two other popular methods to compute the inverse of a matrix. One is similar to the described technique and combines forward elimination and backward substitution. The second technique is based on computing determinants and will be explained in a future revision of this lesson.