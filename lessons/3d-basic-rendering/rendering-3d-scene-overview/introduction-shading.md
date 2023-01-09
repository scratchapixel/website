While everything in the real world is the result of light interacting with matter, some of these interactions are too complex to simulate using the light transport approach. This is when shading kicks in.

![Figure 1: if you look at two objects under the same lighting conditions if these objects seem to have the same color (same hue), but that one is darker than the other, then clearly, how bright they are is not the result of how much light falls on these objects, but more the result of how much light each one of these objects reflects into their environment.](/images/rendering-3d-scene-overview/color-brightness.png?)

As mentioned in the previous chapter, simulating the appearance of an object requires that we can compute the color and the brightness of each point on the surface of that object. Color and brightness are tightly linked with each other. You need to distinguish between the brightness of an object which is due to how much light falls on its surface, and the brightness of an object's color (also sometimes called the color's luminance). The brightness of color as well as its hue and saturation is a color property. If you look at two objects under the same lighting conditions, if these objects seem to have the same color (same [chromaticity](http://en.wikipedia.org/wiki/Chromaticity)), but that one is darker than the other, then clearly, how bright they are is not the result of how much light falls on these objects, but more the result of how much light each one of these objects reflects into their environment. In other words, these two objects have the same color (the same chromaticity) but one reflects more light than the other (or to put it differently one absorbs more light than the other). The brightness (or luminance) of their color is different. In computer graphics, the characteristic color of an object is called **[albedo](http://en.wikipedia.org/wiki/Albedo)**. The albedo of objects can be measured precisely.

<details>
Note that an object **can not** reflect more light than it receives (unless it emits light, which is the case of light sources). The color of an object can generally be computed (at least for diffuse surfaces) as the ratio of reflected light over the amount of incoming (white) light. Because an object can not reflect more light than it receives, this ratio is always lower than 1. This is why the colors of objects are always defined in the RGB system between 0 and 1 if you use float or 0 and 255 if you a byte to encode colors. Check the lesson on [Colors](/lessons/digital-imaging/colors/) to learn more about this topic. It's better to define this ratio as a percentage. For instance, if the ratio, the color, or the albedo (these different terms are interchangeable) is 0.18, then the object reflects 18% of the light it receives back in the environment.
</details>

If we defined the color of an object as the ratio of the amount of reflected light over the amount of light incident on the surface (as explained in the note above), that color can't be greater than one. This doesn't mean though that the amount of light incident and reflected off of the surface of an object can't be greater than one (it's only the ratio between the two that can't be greater than one). What we see with our eyes, is the amount of light incident on a surface, multiplied by the object's color. For example, if the energy of the light impinging upon the surface is 1000, and the color of the object is 0.5, then the amount of light reflected by the surface to the eye is 500 (this is wrong from the point of view of physics, but this is just for you to get the idea - in the lesson on shading and light transport, we will look into what this 1000 or 500 values mean in terms of physical units, and learn that it's more complicated than just multiplying the number of photons by 0.5 or whatever the albedo of the object is).

Thus assuming we know what the color of an object is, to compute the actual brightness of a point P on the surface of that object under some given lighting conditions (brightness as in the actual amount of light energy reflected by the surface to the eye and not as in the actual brightness or luminance of the object's albedo), we need to account for two things:

- How much light falls on the object at this point?
- How much light is reflected at this point in the viewing direction?

Remember again that for specular surfaces, the amount of light reflected by that surface depends on the angle of view. If you move around a mirror, the image you see in the mirror changes: the amount of light reflected towards you changes with the viewpoint.

![Figure 2: to compute the actual brightness of a point P on the surface of that object under some given lighting conditions, we need to account for two things, how much light falls on the object at this point and how much light is reflected at this point in the viewing direction. To compute how much light arrives upon P, we need to sum up the contribution of the light sources (direct lighting) and from other surfaces (indirect lighting).](/images/rendering-3d-scene-overview/shading1.png?)

Assuming we know what the color of the object is (its albedo), we then need to find how much light arrives at the point (let's call it P again), and how much is reflected in the viewing direction, the direction from P to the eye.

- The former problem requires "collecting" or **gathering** light above the surface at P, and is more of a light transport problem. We already explained in the previous chapter how this can be done. Rays can be traced directly to lights to compute direct lighting and secondary rays can be spawned from P to compute indirect lighting (the contribution of other surfaces to the illumination of P). However, while it seems essentially like a light transport problem, we will see in the lessons on Shading and Light Transport that the direction of these rays is defined by the surface type (is it diffuse or specular), and that shaders play a role in choosing the direction of these rays. Note also that, other methods than ray tracing can be used to compute both direct and indirect lighting.
- The latter problem (how much light is reflected in a given direction) is far more complex and it will now be explained in more detail.

First, you need to remember that light reflected in the environment by a surface, is the result of very complex interactions between light rays (or photons if you know what they are) with the material the object is made of. There are three important things to note at this point:

- These interactions are generally so complex that it is not practical to simulate them.
- The amount of light reflected depends on the view direction. Surfaces generally don't reflect incident light equally in all directions. That's not true of perfectly diffuse surfaces (diffuse surfaces appear equally bright from all viewing directions,) but this is true of all specular surfaces and since most objects in the real world have a mix of diffuse and specular reflections anyway, more often than not, light is generally not reflected equally.
- The amount of light redirected in the viewing direction, also depends on the incoming light direction. To compute how much light is reflected in the direction \(\omega_v\) ("v" here is used here for the view and \(\omega\) is the Greek letter omega), we also need to take into account the incoming lighting direction \(\omega_i\) ("i" stands for incident or incoming). The idea is illustrated in figure 3. Let's see what happens when the surface from which a ray is reflected, is a mirror. According to the law of reflection, the angle between the incident direction \(\omega_i\) of a light ray and the normal at the point of incidence, and the direction between the reflected or mirror direction \(\omega_r\) and the normal are the same. When the viewing direction \(\omega_v\) and the reflected direction \(\omega_r\) are the same (figure 3 - top), then we see the reflected ray (it enters the eye). However when \(\omega_v\) and \(\omega_r\) are different (figure 3 - bottom), then because the reflected ray doesn't travel towards the eye, the eye doesn't see it. Thus, how much light is reflected towards the eye depends on the incident light direction \(\omega_i\) (as explained before) as well as the viewing direction \(\omega_v\).

![Figure 3: for mirror surfaces, if the reflected ray and the view direction are not the same, the reflected ray of light is not visible. The amount of light reflected is a function of the incoming light direction and the viewing direction.](/images/rendering-3d-scene-overview/reflection2.png?)

!!!
**Let's summarise**. What do we know?

- It's too complex to simulate light-matter interactions (interactions happening at the microscopic and atomic levels). Thus, we need to come up with a different solution.
- The amount of light reflected from a point varies with the view direction \(\omega_v\).
- The amount of light reflected from a point for a given view direction \(\omega_v\), depends on the incoming light direction \(\omega_i\).
!!!

**Shading, which you can see as the part of the rendering process that is responsible for computing the amount of light reflected from surfaces to the eye (or other surfaces in the scene)**, depends on at least two variables: where light comes from (the incident light direction \(\omega_i\)) and where it goes to (the outgoing or viewing direction, \(\omega_v\)). Where light comes from is independent of the surface itself, but how much light is reflected in a given direction, depends on the surface type: is it diffuse, or specular? As suggested before, gathering light arriving at the incident point is more of a light transport problem. But regardless of the technique used to gather the amount of light arriving at P, what we need, is to know where this light comes from, as in from which direction. The job of putting all these things together is done by what we call a **shader**. A shader can be seen as a program within your program, which is a kind of routine that takes an incident light direction and a view direction as input variables and returns the fraction of light the surface would reflect for these directions.

$$\text{ ratio of reflected light = shader }(\omega_i, \omega_o)$$

Simulating light-matter interactions to get a result is complex, but hopefully, the result of these numerous interactions is predictable and consistent, thus it can be approximated or modeled with mathematical functions. Where are these functions coming from? What are they? How are they found? We will answer these questions in the lessons devoted to shading. Let's only try to get an intuition of how and why this works for now.

The law of reflection for example which we introduced in a previous chapter, can be written as: 

$$\omega_r = \omega_i - 2(N \cdot \omega_i) N$$

In plain English, it says that the reflection direction \(\omega_r\), can be computed as \(\omega_i\) minus two times the dot product between N (the surface normal at the point of incidence) and \(\omega_i\) (the incident light direction) multiplied by N. This equation has more to do with computing a direction than the amount of light reflected by the surface. However if for any given incident direction (\(\omega_i\)), you find out that \(\omega_r\) coincides with \(\omega_v\) (the view direction) then clearly, the ratio of reflected light for this particular configuration is 1 (figure 3 - top). If \(\omega_r\) and \(\omega_v\) are different though, then the amount of reflected light would be 0. To formalize this idea, you can write:

$$\text {ratio of reflected light} = \begin{cases} 1 & \omega_r = \omega_o \\ 0 & \text{otherwise} \end{cases} $$

<details>
This is just an example. For perfectly mirror surfaces, we never proceed that way. The point here is to understand that if we can describe the behavior of light with equations, then we can find ways of computing how much light is reflected for any given set of incident and outgoing directions without having to run a complex and time-consuming simulation. This is really what shaders do: replacing complex light-matter interactions with a mathematical model, which is fast to compute. These models are not always very accurate nor physically plausible as we will soon see, but they are the most practical way of approximating the result of these interactions. Research in the field of shading is mostly about developing new mathematical models that match as closely as possible the way materials reflect light. As you may imagine this is a difficult task: it's challenging on its own, but more importantly, materials exhibit some very different behaviors thus it's generally impossible to simulate accurately all materials with one single model. Instead, it is often necessary to develop one model that works for example to simulate the appearance of cotton, one to simulate the appearance of silk, etc.
</details>

What about simulating the appearance of a diffuse surface? For diffuse surfaces, we know that light is reflected equally in all directions. The amount of light reflected towards the eye is thus the total amount of light arriving at the surface (at any given point) multiplied by the surface color (the **fraction** of the total amount of incident light the surface reflects in the environment), divided by some normalization factor that needs to be there for mathematical/physical accuracy reason, but this will be explained in details in the lessons devoted to shading. Note that for diffuse reflections, the incoming and outgoing light directions do not influence the amount of reflected light. But this is an exception in a way. For most materials, the amount of reflected light depends on \(\omega_i\) and \(\omega_v\).

The behavior of glossy surfaces is the most difficult to reproduce with equations. Many solutions have been proposed, the simplest (and easiest to implement in code) being the Phong specular model, which you may have heard about.

<details>
The Phong model computes the perfect mirror direction using the equation for the law of reflection which depends on the surface normal and the incident light direction. It then computes the deviation (or difference) raised to some exponent, between the actual view direction and the mirror direction (it takes the dot product between these two vectors) and assumes that the brightness of the surface at the point of incidence, is inversely proportional to this difference. The smaller the difference, the shinier the surface. The exponent parameter helps control the spread of the specular reflection (check the lessons from the Shading section to learn more about the Phong model).
</details>

However good models follow some well know properties that the Phong model doesn't have. One of these rules, for instance, is that the model **conserves energy**. The amount of light reflected in all directions shouldn't be greater than the total amount of incident light. If a model doesn't have this property (the Phong model doesn't have that property), then it would break the laws of physics, and while it might provide a visually pleasing result, it would not produce a physically plausible one.

<details>
Have you already heard the term **physically plausible rendering**? It designates a rendering system designed around the idea that all shaders and light transport models comply with the laws of physics. In the early age of computer graphics, speed and memory were more important than accuracy and a model was often considered to be good if it was fast and had a low memory footprint (at the expense of being accurate). But in our quest for photo-realism and because computers are now faster than they were when the first shading models were designed), we don't trade accuracy for speed anymore and use physically-based models wherever possible (even if they are slower than non-physically based models). The conservation of energy is one of the most important properties of a physically-based model. Existing physically based rendering engines can produce images of great realism.
</details>

Let's put these ideas together with some pseudo-code:

```
Vec3f myShader(Vec3f Wi, Vec3f Wo) 
{ 
    // define the object's color, roughness, etc.
    ... 
    // do some mathematics to compute the ratio of light reflected
    // by the surface for this pair of directions (incident and outgoing)
    ... 
    return ratio; 
} 
 
Vec3f shadeP(Vec3f ViewDirection, Vec3f Point, Vec3f SurfaceNormal) 
{ 
    Vec3f totalAmountReflected = 0; 
    for (all light sources above P [direct|indirect]) { 
        totalAmountReflected += 
            lightEnergy * 
            shaderDiffuse(LightDirection, ViewDirection) * 
            dotProduct(SurfaceNormal, LightDirection); 
    } 
 
    return totalAmountReflected; 
} 
```

Notice how the code is separated into two main sections: a routine (line 11) to gather all light coming from all directions above P, and another routine (line 1), the shader, used to compute the fraction of light reflected by the surface for any given pair of incident and view direction. The loop (often called a light loop) is used, to sum up, the contribution of all possible light sources in the scene to the illumination of P. Each one of these light sources has a certain energy and of course, comes from a certain direction (the direction defined by the line between P and the light source position in space), thus all we need to do is send this information to the shader, with the view direction. The shader will return which fraction for that given light source direction is reflected towards the eye and then multiply the result of the shader with the amount of light produced by this light source. Summing up these results for all possible light sources in the scene gives the total amount of light reflected from P toward the eye (which is the result we are looking for).

Not that in the sum (line 18), there is a third term (a dot product between the normal and the light direction). This term is very important in shading and relates to what we call the **cosine law**. It will be explained in detail in the sections on Light Transport and Shading (you can also find information about it in the lesson on the Rendering Equation which you will find in this section). For now, you should just know that it is there to account for the way light energy is spread across the surface of an object, as the angle between the surface and the light source varies.

## Conclusion

There is a fine line between light transport and shading. As we will learn in the section on Light Transport, light transport algorithms will often rely on shaders, to find out in which direction they should spawn secondary rays to compute indirect lighting.

The two things you should remember from this chapter are, the definition of shading and what a shader is:

- Shading, is the part of the rendering process that is responsible for computing the amount of light reflected in any given viewing direction. In another word, it is where and when we give objects in the image their final appearance from a particular viewpoint, how they look, their color, their texture, their brightness, etc. **Simulating the appearance of an object requires answering one question only: how much light does an object reflect (and in which directions), over the total amount it receives?**

- Shaders are designed to answer this question. You can see a shader as some sort of black box to which you ask the question: "if this object is made of wood, if this wood has this given color and this given roughness, if some quantity of light impinges upon this object from the direction \(\omega_i\), how much of that light would be reflected by this object back in the environment in the direction \(\omega_v\)?". The shader will answer this question. We like to describe it as a black box, not because what's happening inside that box is mysterious, but more because it can be seen as a separate entity in the rendering system (it serves only one function which is to answer the above question, and answering this question doesn't require the shader to have any over knowledge about the system than the surface property - its roughness, its color, etc. - and the incoming and outgoing direction being considered) which is why shaders in realtime APIs for example (such as OpenGL - but this often true of all rendering systems whether realtime or off-line) are written separately from the rest of your application.

  What's happening in this box is not mysterious at all. What gives objects their unique appearance is the result of complex interactions between light particles (photons) and atoms objects are made of. Simulating these interactions is not practical. We observed though, that the result of these interactions is predictable and consistent, and we know that mathematics can be used to "model", or represent, how the real world works. A mathematical model is never the same as the real thing, however, it is a convenient way of expressing a complex problem in a compact form, and can be used to compute the solution (or approximation) of a complex problem in a fraction of the time it would take to simulate the real thing. The science of shading is about developing such models to describe the appearance of objects, as a result of the way light interacts with them at the micro- and atomic scale. The complexity of these models depends on the type of surface we want to replicate the appearance of. Models to replicate the appearance of a perfectly diffuse and mirror-like surface are simple. Coming up with good models to replicate the look of glossy and translucent surfaces is a much harder task.

  These models will be studied in the lessons from this section devoted to shading.

<details>
In the past, techniques used to render 3D scenes in real-time were very much predefined by the API with little control given to the users to change them. Realtime technologies moved away from that paradigm to offer a more programmable pipeline in which each step of the rendering process is controlled by separate "programs" called "shaders". The current OpenGL APIs now support four of such "shaders": the vertex, the geometry, the tessellation, and the fragment shader. The shader in which the color of a point in the image is computed is the fragment shader. The other shaders have little to do with defining the object's look. You should be aware that the term "shader" is therefore generally used now in a broader sense.
</details>
