The other advantage of ray tracing is that, by extending the idea of ray propagation, we can very easily simulate effects **like reflection** and **refraction**, which are handy in simulating glass materials or mirror surfaces. In a 1979 paper entitled "An Improved Illumination Model for Shaded Display", **Turner Whitted** was the first to describe how to extend Appel's ray-tracing algorithm for more advanced rendering. Whitted's idea extended Appel's model of shooting rays to incorporate computations for reflection and refraction.

![](/images/introduction-to-ray-tracing/boule-neige.png)

In optics, reflection and refraction are well-known phenomena. Although a later lesson is dedicated to reflection and refraction, we will examine what is needed to simulate them. We will take the example of a glass ball with both refractive and reflective properties. As long as we know the direction of the ray intersecting the ball, it is easy to compute what happens to it. Both reflection and refraction directions are based on the normal at the point of intersection and the direction of the incoming ray (the primary ray). To compute the refraction direction, we also need to specify the material's **index of refraction**. Although we said earlier that rays travel in a straight line, we can visualize refraction as the ray being bent. Its direction changes when a photon hits an object of a different medium (and thus a different index of refraction). The science of this will be discussed in more depth later. We are ready to move on as long as we remember that these two effects depend on the normal vector and the incoming ray direction and that refraction depends on the material's refractive index.

Similarly, we must also be aware of the fact that an object like a glass ball is reflective and refractive at the same time. We need to compute both for a given point on the surface, but how do we mix them? Do we mix 50% of the reflection result with 50% of the refraction result? Unfortunately, it is more complicated than that. The mixing of values depends upon the angle between the primary ray (or viewing direction) and both the object's normal and the refraction index. Fortunately for us, however, an equation calculates precisely how each should be mixed. This equation is known as the **Fresnel equation**. To remain concise, all we need to know, for now, is that it exists and will help determine the mixing values.

![Figure 1: Using optical laws to compute reflection and refraction rays.](/images/introduction-to-ray-tracing/reflectionrefraction.gif)

Let's recap. How does the Whitted algorithm work? We shoot a primary ray from the eye and the closest intersection (if any) with objects in the scene. If the ray hits an object that is not diffuse or opaque, we must do extra computational work. To compute the resulting color at that point on, say, the glass ball, you need to compute the reflection color and the refraction color and mix them. Remember, we do that in three steps. Compute the reflection color, compute the refraction color, and then apply the Fresnel equation.

![](/images/introduction-to-ray-tracing/glassball.png)

- First, we compute the reflection direction. For that, we need two items: the normal at the point of intersection and the primary ray's direction. Once we obtain the reflection direction, we shoot a new ray. Returning to our old example, let's say the reflection ray hits the red sphere. Using Appel's algorithm, we determine how much light reaches that point on the red sphere by shooting a shadow ray to the light. That obtains a color (black if shadowed) multiplied by the light intensity and returned to the glass ball's surface.

- Now, we do the same for the refraction. Because the ray goes through the glass ball, it is said to be a **transmission ray** (light has traveled from one side of the sphere to the other; it was transmitted). To compute the transmission direction, we need the normal at the hit point, the primary ray direction, and the material's refractive index (in this example, it may be something like 1.5 for glass material). With the new direction computed, the refractive ray continues to the other side of the glass ball. There again, because it changes medium, the ray is refracted one more time. As you can see in the adjacent image, the direction of the ray changes when the ray enters and leaves the glass object. Refraction occurs every time there's a change of medium, and the two media, the one the ray exits from and the one it gets in, have a different index of refraction. The refraction index of air is very close to 1, and the refraction index of glass is around 1.5. Refraction has, for effect, bent the ray slightly. This process makes objects appear shifted when looking through or at objects of different refraction indexes. Let's imagine that when the refracted ray leaves the glass ball, it hits a green sphere. We computed the local illumination at the point of intersection between the green sphere and refracted ray (by shooting a shadow ray). The color (black if it is shadowed) is then multiplied by the light intensity and returned to the glass ball's surface

- Lastly, we compute the Fresnel equation. We need the refractive index of the glass ball, the angle between the primary ray, and the normal at the hit point. Using a dot product (we will explain that later), the Fresnel equation returns the two mixing values.

Here is some pseudo-code to reinforce how it works:

```
// compute reflection color
color reflectionCol = computeReflectionColor(); 

// compute refraction color
color refractionCol = computeRefractionColor(); 

float Kr; // reflection mix value
float Kt; // refraction mix value

fresnel(refractiveIndex, normalHit, primaryRayDirection, &Kr, &Kt);

// mix the two colors. Note that Kt = 1 - Kr
glassBallColorAtHit = Kr * reflectionColor + Kt * refractionColor;
```

In the code above, we wrote in the comment that `Kt = 1 - Kr`. In other words, `Kr + Kt = 1`. That's because, in nature, light can't be created or destroyed. Therefore if some of the incident light is reflected, what's left of that incident light (the part that hasn't been reflected) is necessarily refracted. If you take the sum of the reflected and refracted light, it equals the amount of incoming light. Typically, the Fresnel equation provides us with a value for `Kr` and `Kt` (and if it does the right thing, their sum should be equal to 1), so you can use the values returned by the function directly. However, this would be enough if we had only one of them. If you had `Kr`, you could get `Kt` (`1 - Kr`). If you had `Kt`, you could get `Kr` (`1 - Kt`).

One last beautiful thing about this algorithm is that it is **recursive** (that is also a curse in a way, too!). In the case we have studied so far, the reflection ray hits a red, opaque sphere and the refraction ray hits a green, opaque, and diffuse sphere. However, we will imagine that the red and green spheres are also glass balls. To find the color returned by the reflection and the refraction rays, we would have to follow the same process with the red and green spheres that we used with the original glass ball: that is, shooting even more reflection and refraction rays into the scene. This is a drawback of the ray-tracing algorithm that can sometimes become a headache. Imagine that our camera is in a box that has only reflective faces. Theoretically, the rays are trapped and will continue bouncing off the box's walls endlessly (or until you stop the simulation). For this reason, we have to set an arbitrary limit preventing the rays from interacting, thus recursing endlessly. Each time a ray is reflected or refracted, its depth is incremented. We stop the recursion process when the ray depth is greater than the maximum recursion depth. Your image won't necessarily look perfectly accurate, but it is better to have an approximate result than no result.
