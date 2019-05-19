---
layout: post
title:  "Scene Editor Progress"
date:   2011-10-25
tags: [panda3d, panda3d scene editor]
---
It’s been a while since my last post as I’ve been busy with work, but I’ve been chipping away at the editor whenever free time is available. I’ve managed to get the entire pipeline working; from creating models in Maya, exporting them to Panda, arranging them in the editor and then getting them to run in my “game”.

Other stuff that I’ve added recently:

* Selected nodes display their bounding box instead of showing up as red
* Ability to delete or duplicate nodes
* Ability to add directional or point lights, as well as empty node paths
* Basic scene graph outliner

In order to visualize non rendered nodes like lights and locators, I have built some proxy models so the user has something to click and interact with in the viewport. There are two styles of proxies I want to support: one behaves just like a normal model in 3D space, while the other appears 2D and maintains its size in screen space. These models are built using LineSegs, and are typically given collision capsules to roughly outline their shape. When made a child of a node they will inherit their parent’s transform and appear the correct shape on screen. Clicking any collision solid under the node will traverse back up the hierarchy and return the node by using getPythonNetTag() – which is obscenely useful. However I’ve had issues in designing both types of proxies, and have still to come to a solution that I’m entirely happy with.

Panda’s built in collision system does not check lines for collisions, so I need to find a different approach in order to have these nodes selectable. Some users have suggested rendering the entire scene to a texture, then doing a 2D lookup to see if the user has clicked on one of the proxies. Although this approach makes a lot of sense for wire objects, it will probably be too slow to render the entire frame buffer out to texture every time the user clicks on the screen. At the moment I’m using collision solids (capsules, mainly) to approximate the shape of the proxy, then scale the radius of the capsule based on its distance to camera to compensate for distance. This maintains the size of the clickable area for the node and stops the lines becoming increasingly difficult to select the further away the editor camera gets. This works until the user scales up the proxy (something I would like to be supported), and then the capsules end up too big and stop the user from selecting other nodes correctly. I could adjust the radius of the solids by using the inverse matrix of the proxy, but this won’t play nicely when the proxy is scaled non-uniformly. I’ll have to come up with a better idea.

For proxies that maintain screen space size (ambient light / point light) I’ve tried parenting the model to Panda’s 2D render node, aspect2D. Unfortunately this means there is no hierarchial relationship between the 3D light and the 2D proxy that represents it, so the proxy won’t move when the light is transformed. To get around this problem we can add a task to execute every frame which will project the node’s 3D position to its equivalent aspect2D position, and move the proxy to that. Unfortunately this doesn’t solve the problem of selecting the node – I would have to create a collision node in the 3D scene anyway, or implement some sort of 2D picking system. The other alternative is to billboard the model and scale it according to distance to camera. This is probably the better solution as the 2D proxy will never be drawn over a node it is behind, although there are still some issues with drawing the selection bounding box with this technique.

Aside from these and a few more issues, I have something that works! In the video below you can see me create a scene and load that up in first person mode (my ‘game’ – which is pretty much just using the very excellent [ODE Middleware](https://discourse.panda3d.org/t/ode-middleware/7436) developed by a member of the Panda community) and walk around in it. Many thanks to my mate Pete for the use of his models and textures. You can find more of his work [here](http://www.peterhanshaw.com/)!

<iframe width="420" height="315" src="http://www.youtube.com/embed/hgOUurD-uoU" frameborder="0" allowfullscreen></iframe>