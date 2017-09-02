# Assignment 1
## Welcome to your new codebase; it'll be home for the next four months!

Assignment 1 was a simple assignment.. that somehow ended up (probably and hopefully) taking far more time than any future assignment will due to being largely administrative in nature.  To put it simply, I had to set up the Visual Studio Solution for the project, set up the Git Repository, and make some minor changes to some of the example shaders provided.  All in all, it was a lot of configuring with very little programmatic objective.

# Purpose

Still, it had its purpose.  One can't be expected to use a new codebase effectively and efficiently without doing some kind of research into it..  And the administrative tasks of setting up the solution introduced me to the various subsystems in the project.  Many of them are obvious about what they do (things like Concurrency and Graphics).  Others are a bit more elusive in their direct purpose and will require some more examining to fully understand.

# Setting Up

Speaking of the Graphics project, a part of the assignment was the create this project from scratch.  Well.. not the code, that was provided.  But the Visual Studio project had to be created and configured.  This involved setting up the various configuration files for the project and adding references to other projects where they were needed.  It was like a scavenger hunt for broken dependencies.  Which projects needed to link to the Graphics project and which projects did the Graphics project need references to as well?

In the end, I settled on a simple approach:  scan through every file quickly and look for include statements.  In the Graphics project, if it included any files in another project I added that project to its references.  Likewise, if any other projects included files from the Graphics project I added the Graphics Project to their references.  This required opening every file in the entire solution..  And there was a lot...  I didn't look beyond the top of the page, though, as I was hoping all the includes would be located where they should be.. and it seemed they were.

I should note, as part of the assignment requirements, that because of this I didn't find any projects that included files from the Graphics project but didn't need a reference to the Graphics static library.  It is possible such projects would exist.. but I feel it is better to include the reference to the project anyways.

My reasoning is as follows:  if Project A depends on Project B, and Project B depends on Project C, Project A transitively depends on Project C.  (Basic Algebra: `A->B->C` therefore `A->C` .)  This hypothetical Project A shouldn't need to add Project C to its references _in visual studio_ because of how Visual Studio handles references to static libraries.  (They are only linked in the end by the project producing an executable.  Static libraries aren't "linked" together in a single library if one depends on the other, but an executable project is capable of determining by "reference chains" which libraries to link against by iterating down the dependency tree.)

Now, suppose we refactor Project A so that it no longer depends on Project B.  For whatever reason, we managed to get rid of that dependency.  How does A know it needs to depend on C?  It won't unless we remember to add a reference to it in the future.  Thus, I feel it is best to add as reference any project which is directly depended on, even if another dependency would bring along the appropriate static library.  This isn't to say that just because Project A uses Project B it should include as references all of Project B's references.  It should only include Project C if it would directly use Project C itself.

# VM Troubles

Now, I feel it is necessary to note that I don't use Windows or Visual Studio personally.  In fact, I normally do my development, when possible, on a Linux machine.  I find it easier to get into a workflow I like and have access to a shell environment I find more palatable.  But this assignment called for Visual Studio so I thought I'd try and complete it as I had similar academic projects in the past: via Virtual Machine.

Firstly, I ran into issues with the only copy of Windows I had from before I switched to Linux:  Windows 7.  The Windows 7 DirectX SDK is so out of date that it was impossible to build the project.  The code base we were required to use depended on (just about) the latest and greatest D3D that Microsoft has to offer.  Because of that.. I needed an upgrade.

So I downloaded Windows 10 Education edition from the University and gave it a shot.  Everything was set up in about a half an hour or so and I was able to build the project.  Finally...  But..  it wouldn't run.  After another hour of investigation and much trepidation I discovered the problem was that the Virtual Box graphics driver, which was responsible for passing through Graphics calls from the VM to the native graphics card and despite being the latest available, was horribly outdated in terms of capabilities.  It only supported up to "Feature Level 9_3" in Direct3D.  That barely gives one access to the programmatic shader pipeline...

... and the code base requires Feature Level 11_0 or higher.  And, thus, it was discovered I wouldn't be using a VM to get any productive work done..  and I had to go find a Computer Lab the next day.  Luckily, there are many available on campus.

# SHADERS!

This was absolutely the most enjoyable part of the project.  Once I was working on a machine with proper graphics card capabilities I was able to fiddle around and make something neat happen.  I should note, I have prior graphics programming experience, having taken another class at the University of Utah on the subject.  In said class I used OpenGL to complete all the assignments.  As such I could, with some small work, render 3D objects in this codebase without much lecture on the subject from the professor.  (I could even write lighting shaders, shadow shaders, and more!)  Additionally, I've made several shaders for Unity, and, thus, am also somewhat familiar with HLSL.

I was ready for this!

And I decided early on I wanted a rotating triangle.  Now, the proper way would be to pass in via a uniform a model-view-perspetive matrix to the shader rather than calculating it on the GPU..  But, since these shaders would only be processing 3 vertices, I decided to generate one on the fly.

The first thing I did was relocate the points of the singular triangle passed in.  They were originally in the shape of a right triangle that took up about an eighth of the screen.  That wouldn't rotate very well..  so some math later, and I managed to write a bit of if statements to relocate each point independently (and in the right winding order) so that they formed an equilateral triangle with its center at (0,0): the middle of the screen.

Following that I created a rotation matrix in the vertex shader itself and used it, along with the convenient "simulation time" variable provided in the example shader, to rotate the triangle around its center.  Having some background knowledge on Graphics programming definitely made this easy.

But we had more to do!  In addition to "animating the vertices positions" somehow.. we also needed to animate the color!  And as a throwback of sorts to my first assignment in the previously graphics class I took last semester, I decided to animate the color through all the possible hues!  I pulled up a very useful resource [located here](http://www.chilliant.com/rgb2hsv.html) which contained various algorithms for converting between HSV and RGB color spaces and copied the "HUEtoRGB" function into my fragment shaders.

Then it was just a matter of tweaking the simulation time value I passed to that function so that I could use it to animate through all possible values for the hue...  and the result?  This amazing piece of work!

![Assignment 1 Gif](/images/assignment1.gif)

Ah, yes.  This was totally worth it.  (It looks much better if you run the application yourself.  I recommend you check out the link below!)  And with that the assignment was complete.

# Expectations

I'm uncertain as to what I expect to get from this class.  I'm already familiar with several of the core topics we were introduced to in this first week.  I'm familiar with graphics programming, albeit not in a Windows environment.  I have a side project on hold where I plan to build a cross platform renderer like the one we have in this codebase.

Additionally, I'm familiar with Cross-Platform development.  In other previous courses here at the University, whilst the Professor required working code on Windows, I developed a large portion of it to run on both Windows and Linux (doing the initial development on Linux and creating proper interfaces so it could compile and run appropriately on both platforms).  The strategies for handling platform specific implementations explained to us in this class are things I have already done in the past.

I do, however, have some insight into future assignments.  Namely involving the shader pipeline.  (These were partially gained by having hung out with the previous class of students a lot, and partially inferred by noticing how the raw shader files **must** be passed through the Preprocessor as it has include statements in it.)  I expect to spend quite a bit of time in this class developing something comparable to Unity's Shader system; i.e. a system where you can write shaders once and they are "compiled" into versions appropriate for each platform.

All in all.. I'm not entirely sure what to expect from this class.  All I know for sure is that I will find out as the time passes and I complete assignments.  Anyways, here's a download link for the executable:

[Windows - Release - Direct3D](https://github.com/CorneliaXaos/EAE6320-WriteUps/releases/download/a1/Assignment1.zip)

~ Cornelia Schultz
