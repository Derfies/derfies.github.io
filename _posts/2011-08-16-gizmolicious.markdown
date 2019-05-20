---
layout: post
title:  "Gizmolicious"
date:   2011-08-06
tags: [panda3d, tech art]
---
I was a little late in getting this blog started, so I might have to back-date a few posts to cover what I’ve been doing so far on this topic.

For the past couple of months I’ve slowly been chipping away at a project using Panda3D, an engine I decided to use due to its extensive Python integration. After a few dismal attempts in learning the api I decided to try and make something useful, and started work on recreating the manipulators from Maya. I’m still ironing out some bugs, but I’ve got something that looks and behaves just like the real thing.

Luckily Panda makes setting things up ridiculously easy; I’ve had no trouble writing basic controls for mouse picking, marquee selection and Maya-style camera controls. The shapes that make up the gizmos are created using Panda’s geometric classes – they aren’t models loaded from file.

I’ll be making another release soon, but you’re interested in Python and have the Panda3D sdk installed, you can get the first release here.