---
layout: post
title:  "Procedural Generation Rant"
date:   2020-03-28 21:01:00
tags: [procedural generation, game dev]
---
It's been a long time since I made a post and after looking out the window it 
might be a while before normal life resumes, so why not write about a subject 
that's been on my mind for over a decade?

I've been interested in procedural generation since before I learned how to 
code. My first project with Python I tried to generate random maps out of prefab 
tiles with Pygame. I failed miserably, but the the concept behind it has stuck 
with me and I've always end up coming back to it. During the years I spent 
working in the video game industry I produced a number of tools for automated 
content generation but these always took input from an artist - they never 
generated something from absolutely nothing. I can understand the urge to make a 
"magic box" where nothing but perfectly curated content comes from.

That being said, I find myself getting frustrated with how procedural content 
generation is typically used in the game industry these days. There seems to be
a tendency to opt for breadth instead of depth - I would hazard to guess this is 
done for a few reasons:

## Bang For Buck

If you've taken the time to develop a procedural content generation tool then it 
makes sense you'll want to leverage it as much as possible. I can also 
appreciate where the "more is better" psychology comes from; seeing tens of 
pretty assets generated before your very eyes is pretty amazing. Why stop there
when you can make thousands?

## Marketing

Along with the above comes with bragging rights. Big numbers - or even 
mind-bogglingly huge numbers - are kinda impressive and something to talk about.
They can form a nice "back of the box" story to assist with the marketing of a 
title. One can look no further than No Man's Sky and its 18 quintillion planets; 
the procedural generation was part of the core marketing campaign.

## Curbing Consumer Disappointment

I suppose if you were to make an engine that can produce an infinite amount of
something and used that to market the game - but didn't actually *give* the 
consumer this magical box you hyped, they'd quietly get the shits. That makes 
sense - you paid for the game, why not show them the magnificence of the 
generator in its entirety?

Personally I find this method of procedural generation, ironically, limiting. 
Other writers have done a better job at explaining why this is but I find Kate
Compton summarised it succinctly with her [oatmeal analogy.](https://galaxykate0.tumblr.com/post/139774965871/so-you-want-to-build-a-generator)
It highlights that even though we might have a fantastic amount of unique and 
varied content this doesn't necessarily mean what we've produced is particularly
interesting or deep. Funnily enough a common complaint from people reviewing 
such a system is that once they've seem one asset, you've seen them all. The 
generative algorithm and its limits become obvious to the player and they can 
almost start to see what the system may throw at them in the next iteration. If 
the grass is green on this planet and red on the next planet, it comes as little  
surprise that it will be a different colour again on another planet.

Another issue is that techniques for generating art are much further ahead than 
techniques for generating story or game mechanics, or developers choose to 
invest in art generation exclusively. This leaves most developers with a glut of 
art, but because there's no meaningful story or gameplay to reference it the 
experience seems hollow. For example, let's imagine that our generative system 
creates a tower in an empty field. The system may identify this as a place of 
interest and put some sort of reward at the top like a unique item or some 
in-game currency, or alternatively, nothing. Even in the best case scenario the 
reward feels empty. The tower stands in the world as it does in the field, 
unreferenced and unnoteworthy; npcs don't reference it, no story includes it. 
Its existence in the game world is shallow and perfunctory. Once the player 
finds the tower and climbs it, the tower's story is over.

Some attempts of infinite content generation work better than others, typically 
when the interplay of content gels well with other systems. Invisible Inc feels 
fresh and interesting with each map iteration because the layout of the space is 
intrinsically linked to the movement mechanics. Random dungeon layouts in Diablo 
are less interesting as combat generally proceeds in a similar fashion 
regardless of dungeon layout. The much touted Spelunky uses what some consider the
gold standard of procedural generation, a cleaver mix of generated and
hand-crafted content (and one of the main inspirations for my own project).

I've started working on a new procedural toolset that I hope to write about over
the coming weeks. I'm not sure if I can create a system that produces interesting
and deep content but there's definitely some ideas I want to explore.