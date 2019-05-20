---
layout: post
title:  "New Project"
date:   2011-09-20
tags: [panda3d, panda3d scene editor]
---
I’ve started working on a new project, one of the main reasons I wanted to start this blog in the first place. Since starting off in this industry as an environment artist I’ve always loved the idea of building a space and then exploring and interacting with it. I’ve also been looking to start on something which would allow me to work on something with both artistic and technical elements.

The idea for my project is to use the Panda3D engine to build a scene editor, then using that to build an interactive space. Panda also has a web browser plugin, so I can publish these spaces on this website so people can explore these spaces I create. I figure that this ties a number of areas I want to continue pushing forward – my Python skills, my ability as an artist and a tools programmer.

The idea of developing a complete game editor is not something I’m entertaining at the moment; all I require is to be able to drop models and lights into a scene and then be able to serialze that out to xml. Running the scene on the game side would use the same loader code as the editor.

So far I have some basics working:

* Mock-up UI done in wxPython, with nested Panda viewport
* Adding models from file
* Adding point lights
* Selection and transform gizmos
* Saving / loading scenes to / from file
* Toolbar icons from the awesome Fugue set

With the editor in this basic form, the next step will be to create a simple environment with it. I’m planning on implementing the excellent [ODE Middleware](https://discourse.panda3d.org/t/ode-middleware/7436) written by one of the Panda community members to take some of the load off the player camera, controls and movement so I can concentrate on the editor and making scenes with it.

![My helpful screenshot](/assets/images/pandaEditorUi1.jpg)