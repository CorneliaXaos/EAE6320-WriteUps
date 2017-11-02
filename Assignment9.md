# Assignment 9
### 3D EZ

The purpose of this assignment was to add "real" 3D support to our game engine.  The previous assignment laid the foundation, but the meshes we could create before were restricted to a two-dimensional space.. this assignment changed all that!

A lot of little things had to be tweaked, some of which included the following:

* Adding a Physics Project (for basic Rigid Body support.)
* Implementing a Camera and how to submit one to Graphics.
* Upgrading the Mesh classes to include a third dimension.
* Adding a depth buffer/depth stencil to the mesh classes.
* Refactoring the ExampleGame project to use the new code.
* Updating the shader code to properly transform vertices.

## Purpose

To finally have an engine!  A usable engine!  A workable engine!  With this you CAN make a game!  Albeit, it might be a little hard to load in meshes.. and there's no collision detection.. but there's nothing stopping someone from making a simple 3D game with this engine!

## The Assignment

As I mentioned, there were a lot of little things that had to be changed.  Firstly, the old Mesh classes were designed for 2D (as a stepping stone to 3D).  They had to be updated to expect three-dimensional data.  This involved changing quite a few classes, but the changes were relatively small..  It was mostly "dispersed work".

A sizable portion of the assignment went into updating the 'Game -> Graphics -> Shader' pipeline.  This mainly involved creating the Camera implementation for modeling the "observer" of the game.  New Graphics functions had to be created for submitting the camera so that that data could be delivered to shaders.

Finally, the rest of the assignment tended to take place in the ExampleGame project.  Mostly, new meshes had to be created (one of them actually being 3D!) and the new submission functions and such had to be utilized to pass the data appropriately.  I also spent a small amount of time implementing some fun camera controls..  I think it turned out quite nicely.

## Coordinate Spaces

Perhaps the hardest thing for someone new to graphics programming to wrap their heads around would be coordinate frames.  There are five coordinate frames commonly used in Graphics:

* Model Space
* World Space
* View Space
* Clip Space
* Screen Space

(These are the terms I prefer to use for them, though they go by other names as well.)  These five coordinate frames form a chain that all vertex data is passed through before its displayed on screen.  (Some specialized shaders might not send vertex data through all of these, but in the general use case, a mesh will go through all five of these.)  I won't go into much details about them here.. as it becomes difficult to describe them using words alone, but here is a short description of each.

### Model Space

Model Space is specifically the coordinate frame the mesh was built in.  It is used to locate the vertices of the mesh relative to each other by orienting them relative to the origin: (0, 0, 0).  You can think of this as "packaging" the vertices for use in the next coordinate frame. 

### World Space

This coordinate frame contains everything relative to some world origin.  We position our meshes in the world by placing the "packages" built in the previous space throughout it using matrix transformations.  This allows us to create diverse scenes without having to create a "one giant model" mesh in which all the vertices in it are relative to the world origin.  Not only is the "one giant model" approach very inefficient, it makes it very, very difficult to move things around since they'd all be the same model.

### View Space

This is a rotation or translation of World Space such that the origin is now located "between the viewer's eyes".  You could make it so the "viewer" or the camera is located at world origin, but then you'd have to move everything around instead of just moving a camera around.  But if you have any understanding of physics, you'd realize that these are two complimentary reference frames so by moving a single camera around you move everything else the **opposite** way.

### Clip Space

This is perhaps the hardest space for people to grasp, and this is because of the unique problem of 3D graphics:  rendering a 3D image to a 2D screen.  Clip Space is exactly how that is accomplished.  To sum it up briefly, graphics cards only render a small rectangular area.  The only vertices rendered are usually inside of a rectangle that goes from -1 to +1 in all dimensions x, y, and z.  Graphics cards "clip" off everything outside those ranges.  (Hence "clip" space.)

The problem is, our objects are usually spaced further than one unit from the camera...  To make matters worse, things in a 3D space tend to look smaller the further from the viewer they are.. so how do we go from view space to clip space?  By using a projection transformation.

To sum it up briefly (again), a projection transformation moves things from View Space to Clip space.  It places bounds on the minimum and maximum viewing distances of objects and how much the viewer can see to the left, right, up, or down.  It performs some very important math so that the viewer sees this limited portion of world.  If using the Perspective Projection the result imitates how our vision works with things appearing smaller at further distances and parallel lines converging on specific points.

### Screen Space

The transformation from Clip Space to Screen Space is handled internally by the Graphics card, but, essentially, the -1 to +1 cube is flattened into a square to be transferred to the screen.  It's not actually a cube, though.  Rather, the -1 to +1 "x" values correspond to all points horizontally (from left to right) across the viewport (which may or may not be the entire screen) while the -1 to +y "y" values correspond to all points vertically (from bottom to top) across the viewport.  However, the actual coordinates for "points" in screen space vary between hardware technology.  A common system, however, maps the point (-1,+1) in Clip Space to (0,0) in Screen Space (this puts the Screen Space origin in the upper left of the viewport).

(End: Wall of Text. :P )

### To The Shader!

So, how does all that get accomplished in the Vertex Shader?  Well.. we know that the vertices passed into vertex shader are in Model Space and the vertex passed out needs to be in Clip Space.. so.. we obviously need to perform 3 transformations to convert the Model Space vertex from Model Space, to World Space, then View Space, and finally Clip Space.  There are a few ways to do this, but one way is to pass in to the shader the three required transformations.  And how does that look in my code?  Well..  I have a helper function so it's as simple as:

``` glsl
o_position = MODEL_TO_CLIP(vec4(i_position, 1));
```

That wonderful little function macro `MODEL_TO_CLIP` unwraps behind the scenes into the appropriate, platform specific multiplication code to transform `i_position` into Clip Space.

## Wrapping Up

And that sums it all up.  As usual, a build can be found below along with a wonderful gif of just what is possible in the Engine at this point in time.

![Assignment 9 gif](images/a09/assignment9.gif)

(Again, mind the quantization.  It looks much better if you run the program yourself.)

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a9/Assignment9.zip)

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
* Translate the Cube
  * The **Up** and **Down** arrow keys will translate along the World Space y-axis.
  * The **Left** and **Right** arrow keys will translate along the World Space x-axis.
  * The **Page Up** and **Page Down** keys will translate along the World Space z-axis.
* Reset the Camera
  * Press **R**!
  
Enjoy!
