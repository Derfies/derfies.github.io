---
layout: post
title:  "Gizmo Update"
date:   2011-08-16 15:51:31
tags: [panda3d, tech art]
---
After a few long nights coding I think I'm about ready for another release of my gizmo project. I'll be writing up a full post about it later, but if you're keen to give it a go you can download the code [here](http://kurohyou.p3dp.com/Gizmos_v1.1.zip) (you will need [Panda3D](https://panda3d.org) installed to run). Or you can watch this thrilling demo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Y_9hUJRgY-8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Version 1.1 has the following fixes and features:

* Fixed intra module importing
* Moved constants out of __init__.py and into constants.py
* Fixed local transforming for rotation / scale
* Added "complementary" scaling when ctrl-clicking an axis of the scale gizmo
* Added middle mouse functionality to continue transforming
* Fixed bug where moving the mouse from the rotation gizmo would stop transforming
* Fixed bug where quickly rotating in camera axis would spin the gizmo wildly
* Added support for attaching multiple nodes
* Added marquee selection for demo
* Fixed scale gizmo appearance during transform
* Added concept of default axis
* All transformations now done with matrices
* General code cleanup