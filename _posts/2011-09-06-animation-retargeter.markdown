---
layout: post
title:  "Animation Retargeter"
date:   2011-09-06 17:07:50
tags: [tools]
---
One of the last projects I worked on required a massive animation component, one that would most likely require outsourcing to another studio. Having followed this course of action on a previous project, it seemed like a lot of time and energy (and money) wasted on getting very similar animations authored across all the different rigs. Finding a retargeting solution became a priority.

The brief was to create an animation retargeting solution that would retain the animator's keys and was fully automated. Off the shelf solutions like Motion Builder were considered but for pipeline and cost reasons, in addition to animator's preferences, the decision was made not to use it and instead build an in-house solution integrated into 3ds Max.

I decided to use Python over Maxscript early on as there was going to be a lot of animation data being read and written, so I had concerns about its speed. In addition, having the retargeter run as a separate process became useful if Max froze up or stopped working for any reason; we could detect that in Python and kill the process before starting up another one.

![Animation Retargeter UI](/assets/images/animationRetargeterUi.jpg)

One of the other major considerations was that we needed to support complex interactions between actors like grapples and throws. This is where things start to get really tricky, as an actor's motion is likely to be influenced by another actor's body size and shape.

The game I developed this for was unfortunately cancelled before completion. You can however see some of the gameplay [here](https://www.wwe.com/videos/game-trailer-cancelled-wwe-brawl-video-game).
