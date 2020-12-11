---
title:  "Taking a different approach"
date:   2020-12-11 15:44:24 +0100
categories: blender
---
## New conzept

It has been a while since the last post and I need to iterate on some things.  
First of all, further research into the Blender and Cycles source code brought us to question our original conzept for our project. As a reminder we are trying to achieve an incremental render process for viewport rendering using Cycles. Spesifically we want to implement a logic that detects the most affected regions of a viewport by common transformations and only rerender those immediatly, instead of the whole image.

Originally we planned to change the drawing pipeline and the transform pipeline of blender to achieve this, but now we think that we only need to change Cycles code. Cycles has all the necessary information on transformations through the dependancy graph. This approach has the benefit that Cycles is written mostly in C++, which we are more experienced with than plain C which is the main language for Blender.  
See these [answers](https://devtalk.blender.org/t/help-me-understand-how-blender-calls-cycles/16445) in the Cycles section of the Blender development forum, which support this approach.

## New development environment

In addition to the changes to our conzept, I changed my development environment. I moved from Visual Studio to Visual Studio Code as my editor. It is a more lightweight program than VS and with the right plugins you get IntelliSense and language support just as good as in VS.  
See this [wiki post](https://wiki.blender.org/wiki/Developer_Intro/Environment/Portable_CMake_VSCode) for information on how to set it up.

I also use the [ninja](https://ninja-build.org/) build tool now, to speed up my blender builds. It is very straightforward to use and a little faster than plain CMake. To get IntelliSense to work with the Blender and Cycles source code completely in VS Code you need the **compileCommands.json** from the builder.  
Since its not explained very well anywhere how to get this file with your blender build here are some instructions:  

* After you built blender from source with the make tool, go to the build folder.  
* In there you will find a **CMakeCache.txt** file.  
* Open that file and add the following lines:

```Cmake
//Enable/Disable output of compile commands during generation.
CMAKE_EXPORT_COMPILE_COMMANDS:BOOL=YES

//ADVANCED property for variable: CMAKE_EXPORT_COMPILE_COMMANDS
CMAKE_EXPORT_COMPILE_COMMANDS-ADVANCED:INTERNAL=1
```

* Finally locate the **rebuild.cmd** file in the same folder and run it to rebuild blender with the new parameters. 
* After the build is finished you should get the **compileCommands.json** in the same location of the build folder. Works just as well with ninja.