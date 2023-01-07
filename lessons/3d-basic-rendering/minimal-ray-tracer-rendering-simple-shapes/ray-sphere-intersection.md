Intersecting a ray with a sphere is the simplest form of ray-geometry intersection test, which is why many raytracers show spheres images. It also has the advantage (because of its simplicity) of being very fast. However, to get it working reliably, there are always a few important subtitles to give some attention to.

This test can be implemented using two methods. The first one solves the problem using geometry. The second technique, often the preferred solution (because it can be reused for a wider variety of surfaces called quadric surfaces), uses an analytic (or algebraic, e.g., can be resolved using a closed-form expression) solution.

## Geometric Solution

The geometric solution to the ray-sphere intersection test relies on simple maths. Mainly geometry, trigonometry, and the **Pythagorean theorem**. If you look at figure 1, you will understand that to find the position of the points P and P', which corresponds to the points where the ray intersects with the sphere, we need to find value for \(t_0\) and \(t_1\).

![Figure 1: a ray intersecting a sphere and the various terms we will use to solve the ray-sphere intersection with the geometric and analytic solutions.](/images/ray-simple-shapes/raysphereisect1.png?)

Remember that a ray can be expressed using the following **parametric form**:

$$O+tD$$

Where \(O\) represents the origin of the ray and \(D\) is the ray direction (usually normalized). Changing the value for \(t\) makes it possible to define any position along the ray. When \(t\) is greater than 0, the point is located in front of the ray's origin (looking down the ray's direction); when \(t\) is equal to 0, the point coincides with the ray's origin (O), and when \(t\) is negative the point is located behind its origin. By looking at figure 1, you can see that \(t_0\) can be found by subtracting \(t_{hc}\) from \(t_{ca}\) and \(t_1\) can be found by adding this time, \(t_{hc}\) to \(t_{ca}\). All we need to do is find ways of computing these two values (\(t_{hc}\) and \(t_{ca}\)) from which we can find \(t_0\), \(t_1\), and then P and P' using the ray parametric equation:

$$
\begin{array}{lcl}
P & = & {O+t_{0}D}\\
P' &= & {O+t_{1}D}
\end{array}
$$

We will start by noting that the triangle formed by the edges \(L\), \(t_{ca}\), and \(d\) is a right triangle. So we can easily calculate \(L\), which is just the vector between \(O\) (the ray's origin) and C (the sphere's center). We have yet to learn about \(t_{ca}\), but we can use trigonometry to solve this problem.

![Figure 2: \(\vec{a} \cdot \vec{b} = |a||b|\cos\theta\).](/images/ray-simple-shapes/impsurf-proj-vectors.png?)

We know \(L\), and we know \(D\), the ray's direction. We also know that the **dot (or scalar) product** of a vector \(\vec{b}\) and \(\vec{a}\) corresponds to projecting \(\vec{b}\) onto the line defined by the vector \(\vec{a}\), and the result of this projection is the length of the segment AB as shown in figure 2 (for more information on the properties of the dot product, check the [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/math-operations-on-points-and-vectors) lesson):

$$\vec{a} \cdot \vec{b} = |a||b|\cos\theta.$$

In other words, the dot product of \(L\) and \(D\) simply gives us \(t_{ca}\). Note that they can only be an intersection between the ray and the sphere if \(t_{ca}\) is positive (if it is negative, it means that the vector \(L\) and the vector \(D\) points in opposite directions. If there is an intersection, it could be behind the ray's origin, but anything that happens behind it is of no use to us). We now have \(t_{ca}\) and \(L\).

$$
\begin{array}{l}
L=C-O\\
t_{ca}=L \bullet D\\
if \; (t_{ca} \lt 0) \; return \; false
\end{array}
$$

There is a second right triangle in this construction which is defined by \(d\), \(t_{hc}\), and the radius of the sphere. We know the radius of the sphere already, and we are looking for \(t_{hc}\), which we need to find \(t_0\) and \(t_1\). To get there, we need to calculate \(d\). Remember that \(d\) is also the opposite side of the right triangles defined by \(d\), \(t_{ca}\), and \(L\). The Pythagorean theorem says that:

$$opposite \; side^2+adjacent \; side^2=hypotenuse^2$$

We can replace the opposite side, the adjacent side, and the hypotenuse respectively by \(d\), \(t_{ca}\) and \(L\), and we get:

$$
\begin{array}{l}
d^2+t_{ca}^2=L^2\\
d=\sqrt{L^2-t_{ca}^2}=\sqrt{L \bullet L - t_{ca} \bullet t_{ca} }\\
if \; (d &lt; 0)\; return \; false
\end{array}
$$

Note that if \(d\) is greater than the sphere radius, the ray misses the sphere, and there's no intersection (the ray overshoots the sphere). We finally have all the terms we need to calculate \(t_{hc}\). We can use the Pythagorean theorem again:

$$
\begin{array}{l}
d^2+t_{hc}^2=radius^2\\
t_{hc}=\sqrt{radius^2-d^2}\\
t_{0}=t_{ca}-t_{hc}\\
t_{1}=t_{ca}+t_{hc}
\end{array}
$$

In the last paragraph of this section, we will show how to implement this algorithm in C++ and make a few optimizations to speed things up.

## Analytic Solution

Remember that a ray can be expressed using the following function: \(O+tD\) (equation 1) where \(O\) is a point and corresponds to the origin of the ray, \(D\) is a vector and corresponds to the direction of the ray, and \(t\) is a parameter of the function. By varying \(t\) (which can be either positive or negative), we can calculate the position of any point on the line defined by the ray origin and direction. When \(t\) is greater than 0, then the point on the ray is in the "front" of the ray's origin. The point is behind the ray's origin when \(t\) is negative. When \(t\) is exactly 0, the point and the ray's origin are the same.

The idea behind solving the ray-sphere intersection test is that spheres, too, can be defined using an algebraic form. The equation for a sphere is:

$$
\begin{array}{l}
x^2+y^2+z^2=R^2
\end{array}
$$

Where x, y and z are the coordinates of a cartesian point and \(R\) is the radius of a sphere centered at the origin (will see later how to change the equation so that it works with spheres that are not centered at the origin). It says that there is a set of points for which the above equation is true. This set of points defines the surface of a sphere centered at the origin and has a radius \(R\). Or, more simply, if we consider that x, y and z are the coordinates of point P, we can write (equation 2):

$$
\begin{array}{l}
P^2-R^2=0
\end{array}
$$

This equation is typical of what we call in Mathematics, and CG is an **implicit function**, and a sphere expressed in this form is also called an implicit shape or surface. Implicit shapes are shapes that can be defined not in terms of polygons connected, for instance (which is the type of geometry you might be familiar with if you have modeled objects in a 3D application such as Maya or Blender) but simply in terms of equations. Many shapes (often quite simple, though) can be defined as a function (cube, cone, sphere, etc.). However simple, these shapes can be combined to create more complex forms. This is the idea behind modeling geometry using blobs, for instance (blobby surfaces are also called metaballs). But before we get too far off course here, let's get back to the ray-sphere intersection test (check the advanced section for a lesson on **Implicit Modeling**).

All we need to do now is to substitute equation 1 in equation 2, that is, to replace P in equation 2 with the equation of the ray (remember that O+tD defines all points along the ray):

$$
\begin{array}{l}
\begin{matrix}{|O+tD|}\end{matrix}^2-R^2=0
\end{array}
$$

When we develop this equation, we get (equation 3):

$$
\begin{array}{l}
O^2+(Dt)^2+2ODt-R^2=O^2+D^2t^2+2ODt-R^2=0
\end{array}
$$

Which in itself is an equation of the form (equation 4):

$$ \begin{array}{l} f(x)=ax^2+bx+c \end{array} $$

with \(a=D^2\), b=2OD and \(c=O^2-R^2\) (remember that x in equation 4 corresponds to \(t\) in equation 3 which is the unknown). Rewriting equation 3 into equation 4 is important because equation 4 is known as a **quadratic equation**. It is a function for which the roots (when x takes a value for which f(x) = 0) can easily be found using the following equations (equation 5):

$$
\begin{array}{l}
f(x)=ax^2+bx+c
\end{array}
$$

Note the +/- sign in the formula. The first root uses the sign +, and the second root uses the sign -. The letter \(\Delta\) (Greek letter delta) is called the **discriminant**. The sign of the discriminant indicates whether there is two, one, or no root to the equation:

- When \(\Delta\) > 0, there are two roots which can be calculated with:$$ \begin{array}{l} \dfrac{-b+\sqrt{\Delta}}{2a}\quad and \quad\dfrac{-b-\sqrt{\Delta}}{2a} \end{array} $$ In that case, the ray intersects the sphere in two places (at \(t_0\) and \(t_1\)).
- When \(\Delta\) = 0 there is one root which can be calculated with:$$ \begin{array}{l} -\dfrac{b}{2a} \end{array} $$ The ray intersects the sphere in one place only (\(t_0\)=\(t_1\)).
- When \(\Delta\) < 0, there is no root at (which means that the ray doesn't intersect the sphere).

Since we have a, b and c, we can easily calculate these equations to get the values for \(t\), which corresponds to the two intersection points of the ray with the sphere (\(t_0\) and \(t_1\) in figure 1). Note that the root values can be negative, meaning that the ray intersects the sphere but is behind the origin. One of the roots can be negative and the other positive, which means that the ray's origin is inside the sphere. There also might be no solution to the quadratic equations, meaning that the ray doesn't intersect the sphere at all (no intersection between the ray and the sphere).

![Figure 3: several cases might be considered when a ray is tested for an intersection with a sphere. It is important to properly deal with cases where the intersection is behind the ray's origin (spheres 3 and 5). These intersections might sometimes be undesirable.](/images/ray-simple-shapes/rayspherecases.png?)

Before we see how to implement this algorithm in C++, let's see how we can solve the same problem when the sphere is not centered at the origin. Then, we can rewrite equation 2 as:

$$
\begin{array}{l}
|P-C|^2-R^2=0
\end{array}
$$

Where C is the center of the sphere in 3D space. Equation 4 can now be re-written as:

$$|O+tD-C|^2-R^2=0.$$

This gives us the following:

$$
\begin{array}{l}
a =D^2=1\\
b=2D(O-C)\\
c=|O-C|^2-R^2
\end{array}
$$

In a more intuitive form, this comes back to say that we can translate the ray by -C and test this transformed ray against the sphere as if it was centered at the origin.

<details>
"Why a=1?" Because \(D\), the ray direction, is a vector that is normally normalized. The result of a vector raised to the power of 2 is the same as a dot product of the vector with itself. We know that the dot product of a normalized vector with itself is 1 and, therefore, a=1\. However, you must be very careful in your code because the rays tested for intersections with a sphere don't always have their direction vector normalized, so you will have to calculate the value for \(a\) (check code further down). This is a pitfall that is often the source of bugs in the code.
</details>

## Computing the Intersection Point

Once we know the value for \(t_0\), computing the intersection position or hit point is straightforward. We need to use the ray parametric equation:

$$P_{hit} = O + t_0 D.$$

```
Vec3f Phit = ray.orig + ray.dir * t;
```

## Computing the Normal at the Intersection Point

![Figure 4: computing the normal at the intersection point.](/images/ray-simple-shapes/impsurf-normal.png?)

There are different methods to calculate the normal of a point lying on the surface of an implicit shape. However, as mentioned in the first chapter of this lesson, one of these methods uses differential geometry, which is mathematically quite complex. Therefore, we will use a much simpler solution instead. For example, the normal of a point on a sphere can be calculated as the point position minus the sphere center (don't forget to normalize the resulting vector):

$$N = ||P - C||.$$

```
Vec3f Nhit = normalize(Phit - C);
```

## Computing the Texture Coordinates at the Intersection Point

Texture coordinates are, interestingly enough, just the spherical coordinates of the point on the sphere remapped to the range [0, 1]. As recalled in the previous chapter and the lesson on Geometry, the cartesian coordinates of a point can be calculated from its spherical coordinates as follows:

$$
\begin{array}{l}
P.x = \cos(\theta)\sin(\phi),\\
P.y = \cos(\theta),\\
P.z = \sin(\theta)\sin(\phi).\\
\end{array}
$$

These equations might look different if you use a different convention. The spherical coordinates \(\theta\) and \(\phi\) can also be found from the point Cartesian coordinates using the following equations:

$$
\begin{array}{l}
\phi = atan(z, x),\\
\theta = acos(\dfrac{P.y}{R}).
\end{array}
$$

Where \(R\) is the radius of the sphere, these equations are explained in the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/spherical-coordinates-and-trigonometric-functions). Sphere coordinates are useful for texture mapping or procedural texturing. The program of this lesson will show how they can be used to draw a pattern on the surface of the spheres.

## Implementing the Ray-Sphere Intersection Test in C++

Let's now see how we can implement the ray-sphere intersection test using the analytic solution. We could use equation 5 directly (you can implement it, and it will work) to calculate the roots. Still, on computers, we have a limited capacity to represent real numbers with the precision needed to calculate these roots as accurately as possible. Thus the formula suffers from the effect of a **loss of significance**. This happens when b and the root of the discriminant don't have the same sign but have values very close to each other. Because of the limited numbers used to represent floating numbers on the computer, in that particular case, the numbers would either cancel out when they shouldn't (this is called catastrophic cancellation) or round off to an unacceptable error (you will easily find more information related to this topic on the internet). However, equation 5 can easily be replaced with a slightly different equation that proves more stable when implemented on computers. We will use instead:

$$
\begin{array}{l}
q=-\dfrac{1}{2}(b+sign(b)\sqrt{b^2-4ac})\\
x_{1}=\dfrac{q}{a}\\ x_{2}=\dfrac{c}{q}
\end{array}
$$

Where the sign is -1 when b is lower than 0 and 1 otherwise, this formula ensures that the quantities added for q have the same sign, avoiding catastrophic cancellation. Here is how the routine looks in C++:

```
bool solveQuadratic(const float &a, const float &b, const float &c, float &x0, float &x1)
{
    float discr = b * b - 4 * a * c;
    if (discr < 0) return false;
    else if (discr == 0) x0 = x1 = - 0.5 * b / a;
    else {
        float q = (b > 0) ?
            -0.5 * (b + sqrt(discr)) :
            -0.5 * (b - sqrt(discr));
        x0 = q / a;
        x1 = c / q;
    }
    if (x0 > x1) std::swap(x0, x1);
    
    return true;
}
```

Finally, here is the completed code for the ray-sphere intersection test. For the geometric solution, we have mentioned that we can reject the ray early on if \(d\) is greater than the sphere radius. However, that would require computing the square root of \(d^2\), which has a cost. Furthermore, \(d\) is never used in the code. Only \(d^2\) is. Instead of computing \(d\), we test if \(d^2\) is greater than \(radius^2\) (which is the reason why we calculate \(radius^2\) in the constructor of the Sphere class) and reject the ray if this test is true. It is a simple way of speeding things up.

```
bool intersect(const Ray &ray) const
{
        float t0, t1; // solutions for t if the ray intersects
#if 0
        // geometric solution
        Vec3f L = center - orig;
        float tca = L.dotProduct(dir);
        // if (tca < 0) return false;
        float d2 = L.dotProduct(L) - tca * tca;
        if (d2 > radius2) return false;
        float thc = sqrt(radius2 - d2);
        t0 = tca - thc;
        t1 = tca + thc;
#else
        // analytic solution
        Vec3f L = orig - center;
        float a = dir.dotProduct(dir);
        float b = 2 * dir.dotProduct(L);
        float c = L.dotProduct(L) - radius2;
        if (!solveQuadratic(a, b, c, t0, t1)) return false;
#endif
        if (t0 > t1) std::swap(t0, t1);

        if (t0 < 0) {
            t0 = t1; // if t0 is negative, let's use t1 instead
            if (t0 < 0) return false; // both t0 and t1 are negative
        }

        t = t0;

        return true;
}
```

Note that if the scene contains more than one sphere, the spheres are tested for any given ray in the order they were added to the scene. The spheres are thus unlikely to be sorted in depth (with respect to the camera position). The solution to this problem is to keep track of the sphere with the closest intersection distance, in other words, with the closest \(t\). In the image below, you can see on the left a render of the scene in which we display the latest sphere in the object list that the ray intersected (even if it is not the closest object). On the right, we keep track of the object with the closest distance to the camera and only display this object in the final image, which gives us the correct result. An implementation of this technique is provided in the next chapter.

![](/images/ray-simple-shapes/impsurf-proj-results2.png?)