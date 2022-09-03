The other advantage of ray-tracing is that, by extending the idea of ray propagation, we can very easily simulate effects **like reflection** and **refraction**, both of which are handy in simulating glass materials or mirror surfaces. In a 1979 paper entitled "An Improved Illumination Model for Shaded Display", **Turner Whitted** was the first to describe how to extend Appel's ray-tracing algorithm for more advanced rendering. Whitted's idea extended Appel's model of shooting rays to incorporate computations for both reflection and refraction.

![](/images/upload/introduction-to-ray-tracing/boule-neige.png)

In optics, reflection and refraction are well-known phenomena. Although a whole later lesson is dedicated to reflection and refraction, we will look quickly at what is needed to simulate them. We will take the example of a glass ball, an object which has both refractive and reflective properties. As long as we know the direction of the ray intersecting the ball, it is easy to compute what happens to it. Both reflection and refraction directions are based on the normal at the point of intersection and the direction of the incoming ray (the primary ray). To compute the refraction direction we also need to specify the **index of refraction** of the material. Although we said earlier that rays travel in a straight line, we can visualize refraction as the ray being bent. When a photon hits an object of a different medium (and thus a different index of refraction), its direction changes. The science of this will be discussed in more depth later. As long as we remember that these two effects depend on the normal vector and the incoming ray direction and that refraction depends on the refractive index of the material we are ready to move on.

Similarly, we must also be aware of the fact that an object like a glass ball is reflective and refractive at the same time. We need to compute both for a given point on the surface, but how do we mix them? Do we take 50% of the reflection result and mix it with 50% of the refraction result? Unfortunately, it is more complicated than that. The mixing of values is dependent upon the angle between the primary ray (or viewing direction) and both the normal of the object and the index of refraction. Fortunately for us, however, there is an equation that calculates precisely how each should be mixed. This equation is know as the **Fresnel equation**. To remain concise, all we need to know, for now, is that it exists and it will be useful in the future in determining the mixing values.

![Figure 1: using optical laws to compute reflection and refraction rays](/images/upload/introduction-to-ray-tracing/reflectionrefraction.gif)

Let's recap. How does the Whitted algorithm work? We shoot a primary ray from the eye and the closest intersection (if any) with objects in the scene. If the ray hits an object which is not a diffuse or opaque object, we must do extra computational work. To compute the resulting color at that point on, say, for example, the glass ball, you need to compute the reflection color and the refraction color and mix them. Remember, we do that in three steps. Compute the reflection color, compute the refraction color, and then apply the Fresnel equation.

![](/images/upload/introduction-to-ray-tracing/glassball.png)

- First we compute the reflection direction. For that, we need two items: the normal at the point of intersection and the primary ray's direction. Once we obtain the reflection direction, we shoot a new ray in that direction. Going back to our old example, let's say the reflection ray hits the red sphere. Using Appel's algorithm, we find out how much light reaches that point on the red sphere by shooting a shadow ray to the light. That obtains a color (black if it is shadowed) which is then multiplied by the light intensity and returned to the glass ball's surface.

- Now we do the same for the refraction. Note that, because the ray goes through the glass ball it is said to be a **transmission ray** (light has traveled from one side of the sphere to the other; it was transmitted). To compute the transmission direction we need the normal at the hit point, the primary ray direction, and the refractive index of the material (in this example it may be something like 1.5 for glass material). With the new direction computed, the refractive ray continues on its course to the other side of the glass ball. There again, because it changes medium, the ray is refracted one more time. As you can see in the adjacent image, the direction of the ray changes when the ray enters and leaves the glass object. Refraction takes place every time there's a change of medium and the two media, the one the ray exits from and the one it gets in, have a different index of refraction. As you probably know the refraction index of air is very close to 1 and the refraction index of glass is around 1.5). Refraction has for effect to bend the ray slightly. This process is what makes objects appear shifted when looking through or at objects of different refraction indexes. Let's imagine now that when the refracted ray leaves the glass ball it hits a green sphere. There again we compute the local illumination at the point of intersection between the green sphere and refracted ray (by shooting a shadow ray). The color (black if it is shadowed) is then multiplied by the light intensity and returned to the glass ball's surface

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

// mix the two color. Note that Kt = 1 - Kr
glassBallColorAtHit = Kr * reflectionColor + Kt * refractionColor;
```

In the code above we wrote in the comment that `Kt = 1 - Kr`. In other words `Kr + Kt = 1`. That's because in nature, light can't be created nor be destroyed. Therefore if some of the incident light is reflected, what's left of that incident light (the part that hasn't been reflected) is necessarily refracted. If you take the sum of the reflected and refracted light, then it is equal to the amount of incoming light. Typically, the Fresnel equation provides us with a value for `Kr` and `Kt` (and if it does the right thing their sum should be equal to 1), so you can use the values returned by the function directly. However, if we had only one of them, technically this would be enough. If you had `Kr` then you could get `Kt`  (`1 - Kr`). If you had `Kt` then you could get `Kt` (`1 - Kr`).

One last, beautiful thing about this algorithm is that it is **recursive** (that is also a curse in a way, too!). In the case we have studied so far, the reflection ray hits a red, opaque sphere and the refraction ray hits a green, opaque, and diffuse sphere. However, we are going to imagine that the red and green spheres are glass balls as well. To find the color returned by the reflection and the refraction rays, we would have to follow the same process with the red and green spheres that we used with the original glass ball: that is shooting even more reflection and refraction rays into the scene. This is a drawback of the ray-tracing algorithm that can turn into a headeach in some cases. Imagine that our camera is in a box that has only reflective faces. Theoretically, the rays are trapped and will continue bouncing off of the box's walls endlessly (or until you stop the simulation). For this reason, we have to set an arbitrary limit that prevents the rays from interacting, and thus recursing endlessly. Each time a ray is either reflected or refracted its depth is incremented. We simply stop the recursion process when the ray depth is greater than the maximum recursion depth. Your image won't necessarily look perfectly accurate, but better having an approximate result, than no result at all.