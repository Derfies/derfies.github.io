---
layout: post
title:  "Procedural Generation Framework Part II"
date:   2020-05-18 17:52:16
tags: [procedural generation]
---
I've made the decision to completely change my framework. Early results seemd
positive but it's quite clear that using the framework is more of a hinderence
than a help. Part of this is because I've abstracted too heavily in some areas,
or abstracted in the wrong way, putting the developer too far away from actually
writing code that generates things. 
- Ended up having to write an awful lot of boilerplate in order to get something
on screen.
- Generators and selectors ended up being tightly coupled anyway, so might as 
well merge these concepts.
- Assets had to be spawned with their own unique generator which felt clunky.
- Difficult to read
- I can tell that I've been writing pipelines and editors too long. 

Heavily inspired by: http://procworld.blogspot.com/2011/03/writing-architecture.html
I'm set off in a new direction, with a few changes:
- No cursor / context system. I'm assuming this is because he's using his own
grammar system and is going for brevity. I'm using python which can compile JIT,
so I'm not so worried about this.
- No grammar. Same reason - I'm using an interpreted language.

Can also subclass and override program behaviour which was some functionality I 
originally wanted but couldn't quite get with the old system.

# Concepts
- Volumes
    - Has transform matrix
    - Has half extents
    
- Programs / Modules
	- Instantiated with a bounding volume
	- Initial bounding volume can be modified / offset from how the parent authored it
	- Purpose is either to instantiate models or create more bounding volumes
	
- Construction Geometry
	- can be selected 
	- selection returns bounding volumes
		- Existing generators fall into this category
		- Column / row / split goes here too

- Instances (models / tiles)
    - Actual renderable assets

# Potential Interface

    class Volume(object):
    
        """All xform manipulation is done in local space."""
        
        def __init__(self, xform):
            self.xform = xform
            
        @property
        def height(self):
            return scale_from_matrix(self.xform)
            
        def translate(self, value, origin=None):
            origin = origin or (0, 0, 0)
            t = translation_matrix(value, origin)
            self.xform = t * self.xform
            
        def rotate(self, value, origin=None):
            origin = origin or (0, 0, 0)
            r = rotation_matrix(value, origin)
            self.xform = r * self.xform
            
        def scale(self, value, origin=None):
            origin = origin or (0, 0, 0)
            s = rotation_matrix(value, origin)
            self.xform = s * self.xform
            
            
    class GeneratorBase(object):
    
        __metaclass__ = abc.ABCMeta
    
        def __init__(self, volume):
            self.volume = volume
            self.run()
            
        @abc.abstractmethod
        def select(self, name):
            """Return new bounding box."""
            
        @abc.abstractmethod
        def run(self, name):
            """Generate new volumes."""
            

    class ModuleBase(object):
    
        __metaclass__ = abc.ABCMeta
    
        def __init__(self, volume):
            self.volume = volume
            
        @abc.abstractmethod
        def run(self):
            """Generator code goes here."""

- Selectors are still here but they're immediately accessible

# Columns example
So now the column example looks a lot more readable and concise (assuming the
DivideX class is converted over to the new system):

    class Columns(ModuleBase):
    
        def run(self):
            for column in DivideX(self.volume, 10).select('all'):
                height = random.randint(0, column.height)
                column.scale((1, height, 1))
                sections = DivideY(column, 1)
                self.instantiate(sections.select('first'), 'pillar_bottom')
                self.instantiate(sections.select('first'), 'pillar_bottom')
                for section in sections.select('remainder'):
                    self.instantiate(section, 'pillar_middle')