## Ray-Box Intersection

![Figure 1: equation of a line. The equation of a line can be written as y=mx+b. For the oblique line whose equation is y=x-1, we have m=1 and b=-1.](/images/ray-simple-shapes/line.png)

In the following example, we will assume that the box is aligned with the axes of our Cartesian coordinate system. Such a box is called an axis-aligned box or an axis-aligned bounding box (AABB). Boxes that are not oriented along the axis of a cartesian coordinate system are called oriented bounding boxes (OBB). Computing the intersection of a ray with an AABB is quite simple. All we need is to remember that a straight line can be defined as:

$$y = mx + b.$$

In mathematics, the \(m\) term is called the slope (or gradient) and is responsible for the orientation of the line, and \(b\) corresponds to the point where the line intersects the y-axis. If this 1D line is parallel to the x-axis, as shown in figure 1 (with the line whose equation is y=2), then \(m\) is equal to 0\. A ray can also be expressed as: 

$$O + Dt.$$ 

Where \(O\) is the origin of the ray, \(D\) is its direction, and the parameter \(t\) can be any real value (negative, positive, or zero). By changing \(t\) in this equation, we can define any point on the line defined by the ray's position and direction. This equation is very similar to the equation of a line. In fact they are the same: replace \(O\) with \(b\) and \(D\) with \(m\). To represent an axis-aligned bounding volume, we need two points representing the minimum and maximum extent of the box (called bounds in the code).

```
class Box3
{
public:
    Box3(cont Vec3f &vmin, const Vec3f &vmax)
    {
        bounds[0] = vmin;
        bounds[1] = vmax;
    }
    Vec3f bounds[2];
};
```

The volume bounds define a set of lines parallel to each axis of the coordinate system, which we can also express using the line equation. Let's say we have a line defined by the equation: 

$$y = B0_x.$$

Where \(B0.x\) is the x component of the bounding box minimum coordinates (that would be `bounds[0].x` in the code). To find where the ray intersects this line, we can write: 

$$O_x + tD_x = B0_x \text{ (eq1)}.$$ 

Which can be solved by reordering the terms: 

$$t0_x = (B0_x - O_x) / D_x \text{ (eq2)}.$$ 

The x component of the bounding volume's maximum extent can be used similarly to calculate the variable \(t1x\). Note that when the values for \(t\) are negative, intersections are "behind" the ray (behind with respect to the ray's origin \(0\) and its direction \(D\)). Finally, if we apply the same technique to the y and z components, at the end of this process, we will have a set of six values indicating where the ray intersects the planes defined by the six faces of the box. 

$$
\begin{array}{l}
t0_x = \frac{(B0_x - O_x)}{D_x}\\
t1_x = \frac{(B1_x - O_x)}{D_x}\\
t0_y = \frac{(B0_y - O_y)}{D_y}\\
t1_y = \frac{(B1_y - O_y)}{D_y}\\
t0_z = \frac{(B0_z - O_z)}{D_z}\\
t1_z = \frac{(B1_z - O_z)}{D_z}\\ 
\end{array}
$$

![Figure 2: a ray intersecting a 2D box.](/images/ray-simple-shapes/2dfig.png)

Note that if the ray is parallel to an axis, it won't intersect with the bounding volume plane for this axis (in this case, the line equation for the ray is reduced to the constant \(b\), and there's no solution to equation 1). The next step is to find which of these six values corresponds to an intersection of the ray with the box (if the ray intersects the box at all). So far, we have just found where the ray intersects the planes defined by each face of the box. We know this can be found using each of the calculated \(t\) values. By looking at figure 2, the logic behind this test will become clear (we will just be considering the 2D case for this example). The ray first intersects the planes defined by the minimum extent of the box in two places: \(t0x\) and \(t0y\). However, intersecting these planes doesn't necessarily mean that these intersecting points lie on the cube (if they don't lie on the cube, the ray doesn't intersect the box). Mathematically, we can find which of these two points lies on the cube by comparing their values: for that, we need to choose the point whose \(t\) is the biggest. In pseudo-code, we can write:

```
t0x = (B0x - Ox) / Dx 
t0y = (B0y - Oy) / Dy 
tmin = (t0x > t0y) ? t0x : t0y 
```

The process to find the second point where the ray intersects the box is similar; however, in that case, we will use the \(t\) values calculated using the planes defined by the maximum extent of the box and select the point whose \(t\) is the smallest:

```
t1x = (B1x - Ox) / Dx 
t1y = (B1y - Oy) / Dy 
tmax = (t1x < t1y) ? t1x : t1y
```

![Figure 3: a ray intersecting a 2D box.](/images/ray-simple-shapes/2dfigmiss.png)

The ray doesn't necessarily intersect the box. Figure 3 shows a couple of cases where the ray misses the cube. Comparing the \(t\) values can help us find these cases. As you can see in the figure, the ray misses the box when \(t0x\) is greater than \(t1y\) or when \(t0y\) is greater than \(t1x\). Let's add this test to our code:

```
if (t0x > t1y || t0y > t1x) return false;
```

Finally, we can extend the technique to the 3D case by computing \(t\) values for the z component and compare them to the \(t\) values we have calculated so far for the x and y components:

```
t0z = (B0z - Oz) / Dz 
t1z = (B1z - Oz) / Dz 
if (tmin > t1z || t0z > tmax) 
    return false 
if (t0z > tmin) tmin = t0z 
if (t1z < tmax) tmax = t1z 
```

Here is a complete implementation of the method in C++. The variables min and max are the minimum and maximum extent (coordinates) of the bounding box:

```
bool intersect(const Ray &r) 
{ 
    float tmin = (min.x - r.orig.x) / r.dir.x; 
    float tmax = (max.x - r.orig.x) / r.dir.x; 
 
    if (tmin > tmax) swap(tmin, tmax); 
 
    float tymin = (min.y - r.orig.y) / r.dir.y; 
    float tymax = (max.y - r.orig.y) / r.dir.y; 
 
    if (tymin > tymax) swap(tymin, tymax); 
 
    if ((tmin > tymax) || (tymin > tmax)) 
        return false; 
 
    if (tymin > tmin) 
        tmin = tymin; 
 
    if (tymax < tmax) 
        tmax = tymax; 
 
    float tzmin = (min.z - r.orig.z) / r.dir.z; 
    float tzmax = (max.z - r.orig.z) / r.dir.z; 
 
    if (tzmin > tzmax) swap(tzmin, tzmax); 
 
    if ((tmin > tzmax) || (tzmin > tmax)) 
        return false; 
 
    if (tzmin > tmin) 
        tmin = tzmin; 
 
    if (tzmax < tmax) 
        tmax = tzmax; 
 
    return true; 
} 
```

Note that depending on the ray direction, tmin might be greater than tmax. Considering that all the logic behind the tests we are doing, later on, relies on \(t0\) being smaller than \(t1\), we need to swap the values of \(t1\) is smaller than \(t0\). ## Optimizing the Code There are a couple of improvements we can make to this code to make it faster and more robust. First, we can replace the swap operation with the following code:

```
if (ray.dir.x >= 0) { 
    tmin = (min.x - r.orig.x) / ray.dir.x; 
    tmax = (max.x - r.orig.x) / ray.dir.x; 
} 
else { 
    tmin = (max.x - r.orig.x) / ray.dir.x; 
    tmax = (min.x - r.orig.x) / ray.dir.x; 
} 
```

However, this code needs to be fixed. When the value for the ray x, y, or z component is 0, it causes a division by zero. The compiler should handle this gracefully and return `+INF` if this happens, so this doesn't sound like a problem at first glance. The problem is that compilers don't differentiate between 0 and -0 (they consider that `-0 == 0` and will return `true`). So if either one of the ray direction's components has a value of -0, the values tmin/tmax will be set to `+INF` and `-INF`, respectively, instead of `-INF` and `+INF`. This is a problem because instead of executing the second block of the if-statement (the value is negative, not positive, so the `if (ray.dir.x >= 0)` should return false), the first block will be executed instead. Consequently, it will fail to detect the intersection (if there's one). To fix the problem, we can replace the ray direction with the inverse of the ray direction:

```
Vec3f invdir = 1 / ray.dir; 
if (invdir.x >= 0) { 
    tmin = (min.x - r.orig.x) * invdir.x; 
    tmax = (max.x - r.orig.x) * invdir.x; 
} 
else { 
    tmin = (max.x - r.orig.x) * invdir.x; 
    tmax = (min.x - r.orig.x) * invdir.x; 
} 
```

When the ray direction is -0, the inverse value of the direction's component will be set to `-INF,` and the test `if (invdir.x >= 0)` will now return false as desired. In the eventuality of testing the intersection of the ray against many boxes, we can save some time by pre-computing the inverse direction of the ray in the ray's constructor and re-using it later in the intersect function. We can also pre-compute the sign of the ray direction, which we can use to write the method in a much more compact form. Here is the final version of the ray-box intersection method:

```
class Ray
{
public:
    Ray(const Vec3f &orig, const Vec3f &dir) : orig(orig), dir(dir)
    {
        invdir = 1 / dir;
        sign[0] = (invdir.x < 0);
        sign[1] = (invdir.y < 0);
        sign[2] = (invdir.z < 0);
    }
    Vec3<T> orig, dir;       // ray orig and dir
    Vec3<T> invdir;
    int sign[3];
};
 
bool intersect(const Ray &r) const
{
    float tmin, tmax, tymin, tymax, tzmin, tzmax;
    
    tmin = (bounds[r.sign[0]].x - r.orig.x) * r.invdir.x;
    tmax = (bounds[1-r.sign[0]].x - r.orig.x) * r.invdir.x;
    tymin = (bounds[r.sign[1]].y - r.orig.y) * r.invdir.y;
    tymax = (bounds[1-r.sign[1]].y - r.orig.y) * r.invdir.y;
    
    if ((tmin > tymax) || (tymin > tmax))
        return false;

    if (tymin > tmin)
        tmin = tymin;
    if (tymax < tmax)
        tmax = tymax;
    
    tzmin = (bounds[r.sign[2]].z - r.orig.z) * r.invdir.z;
    tzmax = (bounds[1-r.sign[2]].z - r.orig.z) * r.invdir.z;
    
    if ((tmin > tzmax) || (tzmin > tmax))
        return false;

    if (tzmin > tmin)
        tmin = tzmin;
    if (tzmax < tmax)
        tmax = tzmax;

    return true;
}
```

The optimized version runs on our machine approximately 1.5 faster than the first version (the speedup depends on the compiler).

<details>
![](/images/ray-simple-shapes/rayboxisect2.png?) The code from this lesson returns intersections with the box either in front or behind the ray's origin. For instance, if the ray's origin is inside the box (adjacent image), there will be two intersections: one in front of the ray and one behind. We know that an intersection is "behind" the ray's origin when \(t\) is negative. When \(t\) is positive, the intersection is in front of the origin of the ray (with respect to the ray's direction).
</details>

Check the file `raybox.cpp` (which you can find in the last chapter of this lesson) for a complete example. 

![](/images/ray-simple-shapes/impsurf-proj-cube.png?)

## Reference

An Efficient and Robust Ray–Box Intersection Algorithm. Amy Williams et al. 2004.