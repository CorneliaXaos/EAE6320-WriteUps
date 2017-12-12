# Final Project
### FINALLY!

For our final project we were allowed to either make a game using our engine or to add a new feature to it.  I don't really consider this engine to be "feature complete" yet so I decided to add a new feature.. or two.  Specifically, I added basic Material support as well as the ability to submit point lights to the Graphics project.  Additionally, I implemented a system that allows the Graphics project to determine the nearest N lights and to submit those to shaders for use in rendering.

## The Final Project

### The Material Classes

I followed a similar design pattern as the other classes I've created this semester (`Effect`, `Sprite` and `Mesh` are good examples).  I created a reference counted asset called `Material` that controls its own creation and destruction.  I did not set it up to be processed from file, however.  Instead, I followed the same construction style as the previously mentioned classes:  I created a `MaterialBuilder` class that facilitates the construction of `Material` objects in memory.

The `Material` class I created is rather simple.  It doesn't encapsulate an `Effect` within it as I wanted the `Material` class to contain data that is agnostic of particular shader code.  I modeled it, somewhat, after the [WaveFront OBJ Material Format](https://en.wikipedia.org/wiki/Wavefront_.obj_file#Material_template_library), but not entirely so.  My `Material` class allows the user to specify up to 8 textures for binding at runtime into the first 8 texture units on the GPU.  Additionally, it allows for specifying up to 8 "Vector4"s, one for each texture.  These values are submitted / bound to the GPU appropriately when the `Material`'s `Bind` function is called.

How Shader code uses these material values is up to the shader itself.  The lighting shader I developed (outlined in more detail below) uses half of the available slots in the material.  The first four texture units contain the ambient map, diffuse map, specular map, and "shininess" map respectively.  The first four "Vector4" values contain multipliers for each of those maps.

### Lighting

Adding lights to the engine was rather simple (with one major oversight that cost a few hours mentioned below).  I created a simple structure used to represent a (Point) `Light` in memory.  It contains a position for the light, a color for the light, and the light's intensity value.

I then created a submission function that allows the user to submit lights to the Graphics renderer.  The renderer queues them up in a vector until they can be processed during the render loop.  It was important to me that the engine be able to handle a certain number of lights for each object.  The renderer we created is a "Forward Renderer" so handling an unbounded number of lights would be difficult.  Therefore, I designed the system so that it would be capable of lighting each `Mesh` with the N nearest lights to that `Mesh`.

Before a `MeshJob`'s `Render` function is called, the `MeshJob` is instructed to `LightMesh`.  It receives a reference to the vector of submitted lights as well as the `ConstantBuffer` object to Update with the appropriate lighting information.  In order to save on memory, the `LightMesh` function sorts the non-const reference to the vector of lights in place using `std::sort`.  The first N elements of this list represent the N nearest lights to the mesh.  These lights are then used to update the `ConstantBuffer`'s data.

### New Shaders!

In order to get lighting working on the GPU I had to create a couple new shaders (as well as update the previous ones to expect vertex normal data).  The two new shaders I created are both named "mesh-blinn-phong.ppsl".  The rendering algorithm they implement is, obviously, [Blinn-Phong Shading](https://en.wikipedia.org/wiki/Blinn%E2%80%93Phong_shading_model).  I did make some modifications to the algorithm, however, that allows it to utilize up to N number of point light sources.

The fragment shader uses my new `Material` system for determining the ambient, diffuse, and specular color of the rendered fragments.  It also allows specifying the shininess of each fragment through a map as well.  The system could easily be expanded to allow specifying a normal map (or any other texture map) for making even more detailed renderings of meshes.

### How It Went

For the most part, writing the new code was simple.  A lot of the tasks I had to do were things I had to do before in previous assignments (creating a new Graphics class, updating the mesh classes to support normal data, adding new graphics submission functions, etc.).  The `RenderJob` system I had made before allowed me to easily add the `LightMesh` function call to the `MeshJob` class and call it in the appropriate place.

There was only one ghastly bit of oversight...  I spent several hours debugging the Lighting `ConstantBuffer` because the number and type of lights being sent to the GPU seemed wrong.  I had determined that the error was somewhere CPU side.. but I couldn't figure out what it was...  And after about 6+ hours of debugging effort I was informed to "let the program run a little bit and set a breakpoint in a certain function".  I didn't even have to do that as that told me that something was accumulating more data than it should.. and I realized I wasn't clearing the submitted lights between frames..  D'oh!  Once I added that call to `std::vector::clear` everything functioned as it should have.

### Shadows?

I was hoping to get to shadows.. but after having spent so much time debugging something that was a slight goof I decided to call it quits with the system I had developed so far.  I wanted shadows, but setting up the class for rendering "FrameBuffers" on both Direct3D and OpenGL would have been too much work for the amount of time I had left..  (Plus, I had other finals to attend to. :P )

## Semester in Review

### The Assignments

The assignments were interesting.  Some of them I enjoyed more than others.  I feel like they aimed to reinforce Engine Best Practices;  that is to say, they aimed to teach design patterns and architecture layouts best suited to working with larger projects.  Additionally, a lot of the assignments focused on "cross-platform" development in some way or the asset build pipeline.

### What I Learned

Mostly what I gained from this class was experience working in and on a larger system.  Though I didn't touch most of the projects, I became familiar with how the engine was laid out and designed.  Additionally, a lot of this class reinforced knowledge that I had from previous courses such as the Graphics class I took last semester.  Finally, the project as a whole has given me some insight into how best to layout and design larger systems like the Graphics renderer.

### Architecture in General

I tend to prefer an "agile" architecture where only what is needed is designed.  That being said, I love architecting and laying out larger systems, and, since flexibility is something that is needed more often than not, I usually design a system more than the bare minimum as this makes making changes later something easier to handle.

All-in-all, this class has reinforced my opinions on setting up a good architecture for a project.  Namely, good architecture tends to be flexible, maintainable, and easily updated or changed while bad architecture tends to be the opposite of that and has code that has no hierarchy or is highly coupled.

## Wrapping Up

Here's the final screencap of the semester:

![Final Project gif](images/final/finalproject.gif)

As you can see, several portions of the scene are now lit.  The wall, floor, and brain creature thing are all being illuminated by the three point lights in the scene.  The cube and the two translucent objects, however, are not being lit.  This was mainly because I would have had to write another shader for the cube as it is being colored not by a material but by its vertex colors, and the translucent objects would be hard to light given that they are translucent and the GPU already has to treat translucent meshes as "special".

And here's the final download link and a recap of the controls:

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/final/FinalProject.zip)

### Controls

You can use the following controls to:

* Translate The Camera
  * **W** and **S** will move the camera forward and backward.
  * **A** and **D** will strafe the camera left and right.
  * **Q** and **E** will strafe the camera down and up.
* Rotate The Camera
  * **I** and **K** will rotate the camera about the x-axis (pitch).
  * **J** and **L** will rotate the camera about the y-axis (yaw).
  * **U** and **O** will rotate the camera about the z-axis (roll).
* Translate the Brain-Creature-Thing...
  * The **Up** and **Down** arrow keys will translate along the World Space y-axis.
  * The **Left** and **Right** arrow keys will translate along the World Space x-axis.
  * The **Page Up** and **Page Down** keys will translate along the World Space z-axis.
* Reset Everything
  * Press **R**!
  
Enjoy!
