## A Minimal Ray-Tracer: Rendering Spheres

In this chapter, we will detail the program's source code that is provided with this lesson. The program is a basic but functional program that accurately renders a scene containing spheres only. Here is a list of the program's main features:

- Camera transformations are supported. In other words, we can render the scene from any viewpoint. To do so, we will use what we learned in the [previous lesson on generating camera rays](/lessons/3d-basic-rendering/ray-tracing-generating-camera-rays/).
- Geometry visibility is correct. If a ray intersects more than one sphere, we display the sphere with the closest intersection distance.
- When an intersection is found, the normal and the textures coordinates of the intersected point on the sphere are calculated. We will use them both to shade the spheres.

The problem we first need to solve is to find a way of supporting different types of geometry in the program. For example, in this program version, we want to render spheres and spheres only, but what if we want to render polygon mesh in the next version? They are different ways of tackling this problem, but it is usually done in C++ by taking advantage of the class inheritance mechanism. First, we generally define a **base class**, which provides a generic definition of the concept of geometry in the program. Then, we can add to this class as many virtual functions as we like (derived classes can overload virtual functions in C++) and customize these methods for each class derived from the base class. For example:

```
class Object
{
public:
    // virtual intersect function needs to be overloaded by the derived class
    virtual bool intersect(const Vec3f &orig, const Vec3f &dir, float &t) const = 0;
    virtual ~Object() {} // virtual destructor
    Object() {} // constructor
};

class Sphere : public Object
{
public:
    ...
    bool intersect(const Vec3f &orig, const Vec3f &dir, float &t) const
    {
        // add the code to compute the intersection of the ray with an object of Sphere
        ...

        return false;
    }
};

int main(...)
{
    Sphere mySphere;
    ...
    if (mySphere.intersect(orig, dir, t) {
        // this ray intersects this instance of the class Sphere
    }
}
```

In the pseudo-code above, the class `Object` is purely virtual. Therefore, a class derived from the class `Object` needs to implement the `intersect()` method. This method is where the ray-geometry intersection code specific to the geometry type represented by the class is added. The example above would be the ray-sphere intersection code we learned about in the second chapter of this lesson.

```
class Sphere : public Object
{
public:
    ...
     bool intersect(const Vec3f &orig, const Vec3f &dir, float &t) const
    {
        float t0, t1; // solutions for t if the ray intersects
#if 0
        // geometric solution
         Vec3f L = center - orig;
         float tca = L.dotProduct(dir);
         if (tca < 0) return false;
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
    ...
};
```

Next time we will have to add another geometry type; all we need to do is create a new class derived from `Object` and add whatever code is needed to calculate the intersection of a ray with this geometry to the `intersect()` method. For example:

```
class TriangulatedMesh : public Object
{
public:
    ...
     bool intersect(const Vec3f &orig, const Vec3f &dir, float &t) const
    {
        // code to compute the intersection of a ray with triangulated mesh
        ...
        return true;
    }
    ...
};
```

Note that we haven't bothered creating a `Ray` class in this program for simplicity. This is only to show you that this is optional. Plus creating a class to group two variables is not particularly justified. So instead, we use the ray origin and direction directly in the `intersect()` method.

Now that we know how to define a sphere, we can create a scene containing a bunch of spheres whose position in 3D space and radius are randomly computed. The spheres are added to a list of objects. The list can contain an object of the type `Object`, but due to the way inheritance works in C++, any instance of a class that is derived from `Object` can be added to this list as well (plus no instance of the class `Object` can be created because the class is purely virtual). In our case, there would be class `Sphere` instances.

```
int main(int argc, char **argv)
{
    // creating the scene (adding objects and lights)
    std::vector&ltstd::unique_ptr&ltObject&gt&gt objects;

    // generate a scene made of random spheres
    uint32_t numSpheres = 32;
    gen.seed(0);
    for (uint32_t i = 0; i < numSpheres; ++i) {
        Vec3f randPos((0.5 - dis(gen)) * 10, (0.5 - dis(gen)) * 10, (0.5 + dis(gen) * 10));
        float randRadius = (0.5 + dis(gen) * 0.5);
        objects.push_back(std::unique_ptr&ltObject&gt(new Sphere(randPos, randRadius)));
    }

    // setting up options
    Options options;
    options.width = 640;
    options.height = 480;
    options.fov = 51.52;
    options.cameraToWorld = Matrix44f(0.945519, 0, -0.325569, 0, -0.179534, 0.834209, -0.521403, 0, 0.271593, 0.551447, 0.78876, 0, 4.208271, 8.374532, 17.932925, 1);

    // finally, render
    render(options, objects);

    return 0;
}
```

We also set the options, such as the image width and height, in the `main()` function of the program and then pass the object list and the options to the `render()` function. As usual, the `render()` function loops over all image pixels and constructs primary rays. The **camera-to-world** matrix transforms the rays' origin and direction.

```
void render(
    const Options &options,
    const std::vector&ltstd::unique_ptr&ltObject&gt&gt &objects)
{
    ...
    float scale = tan(deg2rad(options.fov * 0.5));
    float imageAspectRatio = options.width / (float)options.height;
    Vec3f orig;
    options.cameraToWorld.multVecMatrix(Vec3f(0), orig);
    for (uint32_t j = 0; j < options.height; ++j) {
        for (uint32_t i = 0; i < options.width; ++i) {
#ifdef MAYA_STYLE
            float x = (2 * (i + 0.5) / (float)options.width - 1) * scale;
            float y = (1 - 2 * (j + 0.5) / (float)options.height) * scale * 1 / imageAspectRatio;
#elif

            float x = (2 * (i + 0.5) / (float)options.width - 1) * imageAspectRatio * scale;
            float y = (1 - 2 * (j + 0.5) / (float)options.height) * scale;
#endif
            Vec3f dir;
            options.cameraToWorld.multDirMatrix(Vec3f(x, y, -1), dir);
            dir.normalize();
            *(pix++) = castRay(orig, dir, objects);
        }
    }

    // Save the result to a PPM image (keep these flags if you compile under Windows)
    ...
}
```

<details>
About the MAYA_STYLE if-else statement: recall that when the image is not a square, we need to stretch the screen window by the image aspect ratio. Mathematically this can be done in two ways. First, you can multiply the screen window in x by the image aspect ratio. Let's say, for instance, that the image resolution is 640x480, providing us with an image aspect ratio of 1.33. The screen window is 1.33 in x and 1 in y (we will assume that the field of view does not influence the screen window size here). Note that we could keep the screen window equal to 1 along the x-axis and scale it down along the y-axis by dividing it by the inverse of the image aspect ratio (as shown in the image below).

![](/images/ray-simple-shapes/impsurf-maya-flag.png?)

This changes the framing of the object seen through the camera, but mathematically both options are perfectly valid. It happens that Maya uses the second option. Thus if you want the output of your ray tracer to match a Maya render, you will need to scale the ray's direction along the y direction by the inverse of the image aspect ratio. Otherwise, you can leave the y-coordinates of the ray unchanged and scale the x-coordinate instead by the image aspect ratio:

```
#ifdef MAYA_STYLE 
float x = (2 * (i + 0.5) / (float)options.width - 1) * scale; 
float y = (1 - 2 * (j + 0.5) / (float)options.height) * scale * 1 / imageAspectRatio; 
#elif 

float x = (2 * (i + 0.5) / (float)options.width - 1) * imageAspectRatio * scale; 
float y = (1 - 2 * (j + 0.5) / (float)options.height) * scale; 
#endif
```

When compiling the program, you can activate the first case by adding the option `-DMAYA_STYLE` to your command.
</details>

The ray origin, direction, and object list are passed to the `castRay()` function. This function is not where we loop over all the scene objects. To find if the ray intersects an object, we will use the `trace()` function instead. The function takes, as an argument, the object list and the ray origin and direction once again. The function loops over all the objects in the scene and calls each object's `intersect()` method. For example, if the object is a sphere, it will call the `intersect()` method of the `Sphere` class. If the object is a `TriangulatedMesh` (though we haven't implemented this class yet), the `intersect()` method of the `TriangulateMesh` class will be called. This function returns true if the ray intersects the object and false otherwise. If an intersection is found, \(t\), which is passed to the `intersect()` method, is set with the distance from the ray origin to the intersected point. The `trace()` function is important because this is where we keep track of the object with the closest intersection distance (as the ray may intersect more than one object). The variable `tNear` is first set to a very large number (line 3) and is only updated when we find that the intersection to the tested object is closer to its actual value (lines 7 and 9). If the object is intersected and passes the test of the nearest object found so far, we also keep a pointer to that object (line 8).

```
bool trace(
    const Vec3f &orig, const Vec3f &dir, 
    const std::vector&ltstd::unique_ptr&ltObject&gt&gt &objects, 
    float &tNear, const Object *&hitObject)
{
    tNear = kInfinity;
    std::vector<std::unique_ptr&ltObject&gt&gt::const_iterator iter = objects.begin();
    for (; iter != objects.end(); ++iter) {
        float t = kInfinity;
        if ((*iter)->intersect(orig, dir, t) && t < tNear) {
            hitObject = iter->get();
            tNear = t;
        }
    }

    return (hitObject != nullptr);
}
```

<details>
Note about [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr): `unique_ptr` is what we call a smart pointer. It retains the ownership of an object through a pointer and destroys that object when the `unique_ptr` goes out of scope. The advantage of using `unique_ptr` to manage the dynamically allocated geometry is that, as a programmer, we don't have to worry about freeing the memory held by this object when it goes out of scope. This is done automatically for us by the smart pointer.
</details>

If the function `trace()` returns true in the `rayCast()` function, then the ray intersects the object defined by the variable `hitObject`. We also know the intersection distance `t` to that object. From there, we can calculate the intersection point (line 9 below) and call the `getSurfaceData()` method of the intersected object to get the normal and texture coordinates of the surface at the intersected point (line 12 below). The normal and the texture coordinates are used to shade the point. We can use the normal with the ray direction (we need to invert the direction) in a dot product to calculate what is called a facing ratio. When the normal and the ray direction point in the same direction, the result of the dot product is close to 1. When the normal and the ray direction are perpendicular (or facing away from each other), the dot product is close to or lower than 0. Finally, we can use the texture coordinate to calculate a checkerboard pattern. The color of the object at the intersection point is a combination of the object's color (set randomly when the object is created), the result of the facing ratio, and the pattern:

```
Vec3f castRay(
    const Vec3f &orig, const Vec3f &dir,
    const std::vector&ltstd::unique_ptr&ltObject&gt&gt &objects)
{
    Vec3f hitColor = 0;
    const Object *hitObject = nullptr; // this is a pointer to the hit object
    float t; // this is the intersection distance from the ray origin to the hit point
    if (trace(orig, dir, objects, t, hitObject)) {
        Vec3f Phit = orig + dir * t;
        Vec3f Nhit;
        Vec2f tex;
        hitObject->getSurfaceData(Phit, Nhit, tex);
        // Use the normal and texture coordinates to shade the hit point.
        // The normal is used to calculate a simple facing ratio, and the texture coordinate
        // to calculate a basic checkerboard pattern
        float scale = 4;
        float pattern = (fmodf(tex.x * scale, 1) > 0.5) ^ (fmodf(tex.y * scale, 1) > 0.5);
        hitColor = std::max(0.f, Nhit.dotProduct(-dir)) * mix(hitObject->color, hitObject->color * 0.8, pattern);
    }

    return hitColor;
}
```

This concludes the description of the program's source code. The result of the program can be seen in the image below. On the left is an image of the scene rendered in Maya. On the right is the image produced by our program. The only difference is the color of the objects, which we have yet to bother replicating in the Maya scene. As you can see, the spheres are in the same position in the image, and the pattern is also the same.

![](/images/ray-simple-shapes/impsurf-proj-results.png?)

As usual, you can find the complete source code in the final chapter of this lesson.