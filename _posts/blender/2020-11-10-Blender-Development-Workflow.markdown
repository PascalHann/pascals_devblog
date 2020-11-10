---
title:  "Blender development workflow"
date:   2020-11-10 18:58:24 +0100
categories: blender
---

I had an unusally hard time figuring out the blender development workflow.  
When you clone the blender git repository from [git://git.blender.org/blender.git](git://git.blender.org/blender.git) and follow the instructions on [blender wiki](https://wiki.blender.org/wiki/Building_Blender/Windows) on building blender, you end up with another folder outside the cloned git repo. This "build" folder contains a Visual Studio solution and the runnable blender executable.  
Now, because of the VS solution I was confused into believing I had to work in this folder rather then the cloned blender repository. Going from there, I couldn't imagine how I would get my changes back into the git repository, since the build folder wasn't even tracked by git.

Thankfully, after some posts on different blender forums I got the right answer: One has to work directly in the cloned blender git repository and only use the build folder for debugging/testing purposes. ([Thread on Blender Stack Exchange](https://blender.stackexchange.com/questions/200591/git-flow-when-coding-for-blender))  

With this newfound knowledge I was able to set up a simple fork of the blender source and also found a convinient way to debug the code from within Visual Studio: Just run the blender executable from the build folder, open the blender repository in Visual Studio and navigate to debug > attach process. Now just select the blender executable in the list of available processes and your already debugging!