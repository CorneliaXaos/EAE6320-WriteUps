# Assignment 7
### A Very Short Assignment

So this assignment was really short for me...  it took me less than an hour to do the coding portion!  Why?  Well..  I had already completed the majority of the assignment before we were given it!  This assignment consisted of two parts: updating some code in the Assets project with (provided) new versions, and updating our shaders to be "platform-independent".  Well..  my shaders were already platform independent!

## Purpose

The purpose of the small code fixes were to address some inefficiencies the class identified with the Asset Manager implementation.  It mainly involved rewriting some of our Graphics code and ExampleGame project code to use the refined interface.  Specifically, the refinements fixed issues I pointed out in the previous Write-Up.

As to the purpose of having platform independent shaders.. well..  having platform independent code is kind of a cornerstone of this class.. so it was only a matter of time before this became an official requirement of our solutions!  And, since I had been designing my shaders to be platform independent since around week 2 or 3.. This portion of the assignment was already done for me! :D

## Manager Code

As I mentioned, this was the bulk of the assignment I had to complete.  I needed to update the Assets project with code provided by the professor to fix the interface issues with the Asset Manager.  This resulted in changing the Graphics Project Sprite submission function signatures as well as how the ExampleGame project called those functions.

All in all, this was a very easy thing to accomplish.

## Shaders!

And, again, as I mentioned.. this was already done!  I had been making my shaders platform independent since around Week 3 and I had been doing so by using the power of MACRO MAGIC...  I believe I mentioned in a previous Write-Up that I refer to my shader files as "PPSL" or "PreProcessed Shader Language" files.  This is because they are passed through the C/C++ preprocessor (I believe) as a part of their compilation.  And this is how platform independent shaders are accomplished.

Basically, the appropriate platform macro is defined based on which build we're in.  That is to say, either `EAE6320_PLATFORM_GL` or `EAE6320_PLATFORM_D3D` is defined.  This allows us to use those defines to create "platform independent" macros to do things such as declaring constant buffers or setting up input and output layouts.

### Textures, As An Example

For example, I created a `textures.inc` include file which defines useful macros for declaring and sampling from textures in both OpenGL and D3D.  If I wanted to declare a texture to be sampled, I coudld do the following:

`LayoutSampler2D( diffuse, 0 )`

This lays out a 2D texture that the shader can then sample later.  Specifically, it specifies that the texture should be located in Texture Unit 0 on the GPU, though this could be changed programmatically.  Then, if I wanted to sample that texture, I could do this:

`o_color = Sample2D( diffuse, i_uvs )`

### gl_Position

The only other thing of note is how I handle `gl_Position`.  `gl_Position` is an OpenGL special variable used in the Vertex Shader for outputting the location of a vertex.  I decided to take the easiest route of having a preprocessor define that is executed for the OpenGL Platform:

`#define o_position gl_Position`

This may restrict me to **always** and **only** use o_position for my vertex output.. but I am fine with that.  I will just treat it as a special variable used in my "PPSL".

## Conclusion

So.. that's all folks!  No screenshot this week as nothing visually has changed from the previous project.. still, here is a download for you to play (which should also appear identical to the previous release)!

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a7/Assignment7.zip)
