Besides points, vectors, normals, and matrices, the last helpful technique from linear algebra we will need to render images is expressing vectors in terms of spherical coordinates. We could certainly render images without using them, but you will see that using them simplifies many problems, especially regarding shading. This chapter is also an excellent opportunity to review trigonometric functions.

## Trigonometric Functions

![Figure 1: the sine and cosine function can be used to find the coodinate of P which lies on the unit circle.](/images/geometry/trigonometry.png?)

Rendering computer-generated images is almost entirely a geometric problem, so understanding and using trigonometry for creating such images (and the Pythagorean theorem) would be very hard. Let's start to review the **sine** and **cosine** functions and how angles can be calculated from 2D coordinates. Usually, these functions are defined in regards to the **unit circle** (a circle of radius 1). When we draw a point P on this unit circle, the x-coordinate of the point can be calculated using the cosine of the angle subtended by the x-axis and a line that goes from the origin of the coordinate system to P. This angle is usually called \(\theta\) (the greek letter theta). Similarly, the sine of this angle gives the y-coordinate of the point P. Note that the angle \(\theta\) is defined in **radians**. It will be easier to determine the angles in degrees. Still, we will need to convert them internally to radians to use them in the C++ trigonometric functions: \(\theta_{radians} = {\pi \over 180}\theta_{degrees}\). Remember that a complete turn around the unit circle represents 360 degrees of \(2 \pi\).

![Figure 2: names given to the side of a right-triangle](/images/geometry/triangle.png?)

It is also important to remember that the cosine, sine, and tangent functions are defined from a simple relationship between the edges of a right triangle (right-angled triangle). The tangent formula is interesting because, to come back to our example using the unit circle, it can be calculated using the ratio of y over x. Another handy function in Computer Graphics is the **arctangent**, the tangent inverse function. In other words, if you feed the arctangent function with the result of the tangent function, you get \(\theta\). In programming, you can use the `atan` function, but this function doesn't consider the sign of the parameters x and y. For instance, if P has coordinates (0.707, 0.707), the angle \(\theta\) is \(\pi / 4\). If the coordinates of P are now (-0.707, -0.707), theta should then be \(3 \pi / 4\). But the tan function will calculate the ratio -0.0707/-0.0707, which is 1; the result of the \(\tan\) function for such coordinates will thus be \(\pi / 4\) though this is the angle for the point with coordinates (0.707, 0.707) and not (-0.707, -0.707). This is wrong. To fix the issue, you need to use the C/C++ function `atan2` instead, which takes into account the sign of the point's coordinates in the angle computation (check the documentation on the function for further details). Similarly to `atan2`, you can calculate the inverse function of sine and cosine using arcsine (`sin` in C++) and arccosine (`acos`). Let's summarise all the functions we have talked about so far:

$$
\begin{array}{l}
\sin(\theta)={\text{opposite side} \over \text{hypothenuse}}\\
\cos(\theta)={\text{adjacent side} \over \text{hypothenuse}}\\
\tan(\theta)={\text{opposite side} \over \text{adjacent side}}
\end{array}
$$

$$
\begin{array}{l}
\theta = \text{acos}(P_x)\\
\theta = \text{asin}(P_y)\\ 
\theta = \text{atan2}(P_y, P_x)
\end{array}
$$

Refer to the documentation of these functions to learn what they exactly return. The interesting thing to note is that the angle returned by the `atan2` function is positive for counter-clockwise angles (upper half-plane, y > 0) and negative for clockwise angles (lower half-plane, y < 0). It produces results in the range \([-\pi, \pi]\). Finally, let's finish this quick reminder with the Pythagorean Theorem, which we will also use often (for example, the ray-sphere intersection test, which you can find explained in the Foundations of 3D Rendering section). It says that:

$$hypothenuse^2=adjacent^2 + opposite^2$$

In other words, the square of the hypothenuse length is equal to the sum of the squares of the other two sides of the right triangle (adjacent and opposite).

## Representing Vectors with Spherical Coordinates

![Figure 3: a vector can also be represented by two angles: the vertical angle (in red) \(\theta\) and the horizontal angle (in green) \(\phi\).](/images/geometry/sphericalcoord.png?)

![Figure 4: in the top figure, we are looking perpendicularly to the plane defined by the vector and the up-axis. In the bottom figure, we are looking at the vector from the top. The angle \(\theta\) (top figure) can vary from 0 to \(\pi\) and the angle \(\phi\) (bottom figure) can vary from 0 to \(2\pi\).](/images/geometry/sphericalcoord1.png?)

So far, we have learned how to represent vectors (as in directions) using cartesian coordinates (with three values, one for each axis). It is also possible to represent the same vectors with only two values. One represents the angle between the vector and the vertical axis and the angle between the vector projected onto the horizontal plane and the right vector from the Cartesian coordinate system. In Figure 3, these angles are represented in red and green. The vertical angle is always called \(\theta\) (the greek letter theta), and the horizontal angle (in green) is always called \(\phi\) (the greek letter phi). No matter what you do and what you see in textbooks, we advise you to follow these rules, which is the only convention unanimously followed by the CG community. These angles should be expressed in radians. Note that \(\theta\) lies within the range \([0:\pi]\) while \(\phi\) varies in the range \([0:2\pi]\) (see Figure 4). As such, \(\theta\) and \(\phi\) can also be seen as coordinates and are called **spherical coordinates**. In Figure 4, we can see the vector in 2D view. On top, we are looking perpendicularly to the plane defined by the vector and the up-axis. The bottom figure represents a view from the top. Vr Vu and Vf correspond to the cartesian coordinates of the vector in the Cartesian coordinates systems defined by the right, up, and forward axes. Note that we haven't used the names x, y, and z for the axis for a reason we will explain soon. Also, we have always represented a normalized vector (of unit length), but any vectors of arbitrary length can be represented using spherical coordinates. The formal definition of spherical coordinates includes an additional term (usually denoted r for radial distance) to represent the length of the vector combined with \(\theta\) and \(\phi\), which can also be called the **polar** and **azimuth** angles. Spherical coordinates are just another way of encoding vectors. They make this representation compact as only two numbers are used instead of three (if you don't care about the length of the vector) with the Cartesian coordinates (it can save memory in your program). They will become most useful when we talk about shading. How do we convert a vector represented in Cartesian coordinates to spherical coordinates?

## Conventions Again: Z is Up!

![Figure 5: in mathematics and physics, spherical coordinates are represented in a Cartesian coordinate system where the z-axis represents the up vector.](/images/geometry/sphericalcoord3.png?)

The convention for representing vectors in mathematics and physics is to name the up vector as the z-axis and the right and forward vector, respectively, the x- and y-axis. And to make things easier, the convention is also to use a **left-hand coordinate** system (which you can see in Figure 5). It will likely use this convention if you read an article on spherical coordinates from a reliable wiki. We have already briefly mentioned a z-axis representing the up vector in the chapter on [Coordinate System](/lessons/mathematics-physics-for-computer-graphics/geometry/coordinate-systems). As you can see, this convention is different from the one we usually use (where the up-axis is the y-axis), but unfortunately, this notation is the norm, and we will have to stick to it. Our main point of interest is how this will affect our code. When we study shading, you will see that we use a trick to convert the vectors from world space to a local coordinate system where the normal at the surface of the shaded point represents the up vector (see the next chapter [Creating an Orientation Matrix or Local Coordinate System](/lessons/mathematics-physics-for-computer-graphics/geometry/creating-an-orientation-matrix-or-local-coordinate-system)). However rather than building the matrix to transform vector from whatever space they are into this local coordinate system by copying the tangent (x-axis), the normal (y-axis) and the bi-tangent (z-axis) to the first (right vector), second (up vector) and third (forward vector) row of the matrix as we usually do, we will copy them in this order:

$$\begin{bmatrix}T_x&T_y&T_z&0\\B_x&B_y&B_z&0\\N_x&N_y&N_z&0\\0&0&0&1\end{bmatrix}$$

T, B, and N represent the tangent bi-tangent and normal vectors. Note that we swapped the position of the normal (up vector or y-axis in the conventional coordinate system) and the bitangent (forward vector or z-axis in the traditional coordinate system) in the matrix construction. Let's see how this work. Imagine you have a normal whose coordinates in world space are (0, 1, 0). In other words, it points straight up. Let's construct a matrix using the trick we have just learned where the tangent and bit agent vectors have the coordinates (1, 0, 0) and (0, 0, 1):

$$\begin{bmatrix}1&0&0&0\\0&0&1&0\\0&1&0&0\\0&0&0&1\end{bmatrix}$$

Now imagine you want to transform a vector \(v\) in the local frame represented by this matrix and that the coordinates of this vector are (0, 1, 0). It is parallel to the y-axis in world space. If we apply the matrix-vector multiplication formula, we get:

$$
{
\begin{array}{l}
x = Vx * M_{00} + Vy * M_{10} + Vz * M_{20} = 0 * 1 + 1 * 0 + 0 * 0 = 0\\
y = Vx * M_{01} + Vy * M_{11} + Vz * M_{21} = 0 * 0 + 1 * 0 + 0 * 1 = 0\\
z = Vx * M_{02} + Vy * M_{12} + Vz * M_{22} = 0 * 0 + 1 * 1 + 0 * 0 = 1
\end{array}
}
$$

As you can see, once transformed, the vector has coordinates (0, 0, 1). It is aligned with the up vector, which is represented by the normal, which is also, in this case, the z-axis. So we successfully converted a vector in a coordinate system where the z-axis is the up vector. This concept needs to be clarified, especially if you display the resulting vector in a 3D application where the y-axis is up, and the z-axis is the forward axis. However, the way you look at this is more like a swap of the y- and z-coordinates of the vector.

## Converting Cartesian to Spherical Coordinates

![Figure 6: it is easier to see that Vz is equal to \(\cos(\theta)\) when we rotate the figure by 90 degrees clockwise.](/images/geometry/sphericalcoord4.png?)

For the demonstration, we will assume that the vector is normalized. The illustration on the left in Figure 6 is the same as the top illustration in Figure 4 but with the up vector now represented as the z-axis (in blue). If we rotate the figure by 90 degrees clockwise (on the right of Figure 6), you can see that it looks similar to Figure 1, where the x-coordinates (in Figure 1) were calculated using the formula \(\cos(\theta)\). Applied to the case shown in Figure 6, we can therefore say that Vz is equal to \(\cos(\theta)\) as well (here, Vz is the same as Px in Figure 1). And consequently, the angle \(\theta\) itself can be calculated as the arccosine of the value Vz:

$$\begin{array}{l}V_z = \cos(\theta) \rightarrow \theta = acos(V_z)\end{array}$$

In C++, you will write:

```
float theta = acos(Vz);
```

![Figure 7: computing the angle \(\phi\).](/images/geometry/sphericalcoord5.png?)

Let's now find out how to calculate the angle \(\phi\). Let's now look at Figure 7, the same as the illustration at the bottom of Figure 4, but where the right and forward axes have now been named the x- and y-axis (in red and green). Remember from our quick trigonometric function refresher (at the top of this chapter) that the tangent of an angle can be calculated by taking the ratio of the opposite side (which is Vy in this example) over the adjacent side (Vx) of a right triangle. You may ask why we do not calculate this angle as we did for \(\theta\), where we used the arccosine of the value Vx to find \(\phi\). That is an option, but don't forget that \(\phi\) varies from 0 to \(2\pi\). The advantage of using the tangent rather than the cosine is that the C++ implementation of the function (or rather the `atan2` C++ function) will take into account the sign of its arguments (Vy and Vx) to return an angle that either varies from 0 to \(\pi\) if the vector is in the right part of the unit circle and 0 to \(-\pi\) if the vector is in the left part of the unit circle. As a programmer, you will need to remap this value to the range \([0:2\pi]\) if necessary (look at the end of this lesson for the complete code):

$$tan(\phi)= { V_y \over V_x } \rightarrow \phi = atan({V_y \over V_x})$$

In C++, you will write:

```
float phi = atan2(Vy, Vx);
```

## And Vice Versa: Spherical Coordinates to Cartesian Coordinates

The formula to calculate cartesian coordinates back from spherical coordinates is straightforward:

$$
\begin{array}{l}
x =\cos(\phi)\sin(\theta)\\
y=\sin(\phi)\sin(\theta)\\
z=\cos(\theta)
\end{array}
$$

It is not always easy to remember this formula by heart, but it is always possible to re-write it from simple deductions. We know that the z coordinate of the vector only depends on the angle theta and that \(V_z = \cos(\theta)\). As for the x coordinate, imagine that you want V to have coordinates (1, 0, 0) which are true when \(\theta = \pi / 2\) and \(\phi=0\). We know that \(\sin(\pi / 2)=1\) and \(\cos(0)=1\) thus \(x=\sin(\theta)\cos(\phi)\). The same technique can be used to find y. Here is some C++ code to calculate cartesian coordinates from the two spherical angles:

```
template<typename T> 
Vec3<T> sphericalToCartesian(const T &theta, const T &phi) 
{ 
    return Vec3<T>(cos(phi) * sin(theta), sin(phi) * sin(theta), cos(theta)); 
}; 
```

## More Tricks with Trigonometric Functions

Now that we have explained how to convert from cartesian coordinates to spherical and vice versa, we will show a couple of useful functions that can be used in the renderer to manipulate vectors using both representations.

The first function we will write is calculating \(\theta\) from the cartesian coordinates. Remember that we will use a left-hand coordinate system in which the z-axis is the up vector for spherical coordinates. We have explained in this chapter that we can write:

```
template<typename T> 
inline T sphericalTheta(const Vec3<T> &v) 
{ 
    return acos(clamp<T>(v[2], -1, 1)); 
} 
```

Note that the input vector should be normalized, so the vector's z-coordinates should be in the range [-1:1], but clamping this value is safer.

Next, we will write a function to calculate \(\phi\). We have mentioned in this chapter before that the function `atan` returns a value in the range \([-\pi:\pi]\). So we will need to remap this value in the range \([0:2\pi]\).

```
template<typename T> 
inline T sphericalPhi(const Vec3<T> &v) 
{ 
    T p = atan2(v[1], v[0]); 
    return (p < 0) ? p + 2 * M_PI : p; 
} 
```

It is only sometimes necessary to calculate the angle values from the cartesian coordinates. Sometimes we just want to get the values for \(\cos(\theta)\), \(\sin(\theta)\), \(\cos(\phi)\) or \(\sin(\phi)\). Computing \(\cos(\theta)\) is straightforward (it is very similar to the function sphericalTheta we wrote earlier):

<div name="code" class="code">template<typename T> inline T cosTheta(const Vec3<T> &w) { return w[2]; }</div>

Computing \(\sin(\theta)\) is a bit more complicated. We know that a vector lying on the unit circle has length 1 (unit length). We also know (Pythagorean theorem) that for such vector, we can write \(V_x^2+V_y^2=1\). If \(V_x = \cos(\theta)\) and \(V_y =\sin(\theta)\) we can write:

$$\cos(\theta)^2 + \sin(\theta)^2 = 1 \rightarrow \sin(\theta)^2=1-\cos(\theta)^2$$

We can first write a function that calculates \(\sin(\theta)^2\) and then another one to calculate \(\sin(\theta\)), which returns the square root of the result returned by the first function:

```
template<typename T> inline T cosTheta(const Vec3<T> &w) { return w[2]; } 
```

![Figure 8: the shadow of the yellow vector \((v)\) corresponds to the projection of \((v)\) on the xy plane. This projected vector \((v_p)\) is shorter than a unit length vector, but to normalize it, we can divide it xy coordinates by \(\sin(\theta)\), which we then can use to calculate \(\sin(\phi)\) and \(cos(\phi)\).](/images/geometry/projectvec.gif?)

Computing \(\cos(\phi)\) and \(\sin(\phi)\) is also slightly more complicated. As you can see in Figure 8, even though the vector \(v\) is of unit length in world space, its shadow on the xy plane creates a vector that doesn't lie on the unit circle (unless \(\theta = \pi/2)\). Technically speaking, the shadow of this vector corresponds to **projecting** the vector \(v\) on the xy plane. However, using `atan2` to calculate \(\phi\) only works for the unit length vector. You can also notice that the length of the vector \(v\) projected in the xy plane (\(v_p\)) is directly related to the angle \(\theta\) (Figure 9). For values of \(\theta\) close to 0 or \(\pi\), \(v_p\) is very small (example on the left in Figure 8), and for values close to \(\pi/2\), \(v_p\) is longer (it lies on the unit circle when \(\theta=\pi/2\)).

![Figure 9\. Left: the length of the vector \(v_p\) can be calculated using \(\sin(\theta)\). Right: the x and y coordinates of the vector \(v_p\), once normalized, can be used to calculate phi (using atan2(y, x)).](/images/geometry/projphi.png?)

The value \(\sin(\theta)\) represents the length of the vector \(v_p\) which is the vector v projected on the xy plane (Figure 9a). Dividing the coordinates of \(v_p\) by this length has the effect of normalizing \(v_p\). Once a unit length vector, the x and y coordinates of \(v_p\) can be used to calculate \(\sin(\phi)\) and \(\cos(\phi)\).

```
template<typename T> 
inline T cosPhi(const Vec3<T> &w) 
{ 
    T sintheta = sinTheta(w); 
    if (sintheta == 0) return 1; 
    return clamp<T>(w[0] / sintheta, -1, 1); 
} 
 
template<typename T> 
inline T sinPhi(const Vec3<T> &w) 
{ 
    T sintheta = sinTheta(w); 
    if (sintheta == 0) return 0; 
    return clamp<T>(w[1] / sintheta, -1, 1); 
}
```