---
layout: post
title:  "Attaching Scripts to Objects in Panda3D – Part II"
date:   2012-05-06
tags: [Musings, Panda3D, Panda3D Scene Editor]
---
In my last post I showed how to attach a Python object to a node path in order to create a ‘hook’ in the Panda3D scene graph. In this post I’ll be showing how to dynamically add additional code to that object at runtime. Attaching code to objects in this way will be at the core of offering drag and drop scripting functionality in the same manner as Unity.

Once we have the PandaObject the next thing we could do is to subclass it in order to add additional code. Since I’m building an editor I want to offer the user an easy way to add, remove and combine different scripts for a node path, so in this case we’ll use the PandaObject as a hook only and then ‘hang’ other scripts from it.

![My helpful screenshot](/assets/pandaEditorUi2.jpg)

In the context of our editor, the user will presented with a file browser displaying all the scripts in their project. This hierarchy will be representative of the directory structure on disk, and the user should be able to drag and drop any script onto any node in the scene. This presents an interesting problem as essentially we need to be able to instantiate a class from a file path the user selects at runtime. Thankfully python’s imp module offers some very handy tools for solving this kind of problem.

So now our PandaObject code looks like this:

{% highlight python %}
import os
import sys
import imp
 
from direct.showbase.DirectObject import DirectObject
 
class PandaObject( object ):
 
    def __init__( self, np ):
 
        # Store the node path with a reference to this class attached to it
        self.np = np
        self.np.setPythonTag( 'PandaObject', self )
 
        self.instances = {}
 
    @staticmethod
    def Get( np ):
 
        # Return the panda object for the supplied node path
        return np.getPythonTag( 'PandaObject' )
 
    @staticmethod
    def Break( np ):
 
        # Detach each script from the object
        pObj = PandaObject.Get( np )
        if pObj is not None:
            for clsName in pObj.instances.keys():
                pObj.DetachScript( clsName )
 
        # Clear the panda object tag to allow for proper garbage collection
        np.clearPythonTag( 'PandaObject' )
 
    def AttachScript( self, filePath ):
 
        # If the script path is not absolute then we'll need to search for it.
        # imp.find_module won't take file paths so create a list of search
        # paths by joining the head of the script path to each path in sys.path.
        head, tail = os.path.split( filePath )
        if not os.path.isabs( filePath ):
            srchPaths = []
            for sysPath in sys.path:
                srchPaths.append( os.path.join( sysPath, head ) )
        else:
            srchPaths = [head]
 
        # Import the module. Pass imp.find_module the name of the module
        # and a list of paths to search for it on.
        name = os.path.splitext( tail )[0]
        modDetails = imp.find_module( name, srchPaths )
        mod = imp.load_module( name, *modDetails )
 
        # Get the class matching the name of the file, attach it to
        # the object
        clsName = name[0].upper() + name[1:]
        cls = getattr( mod, clsName )
 
        # Save the instance by the class name
        self.instances[clsName] = cls( self.np )
 
    def DetachScript( self, clsName ):
 
        # Remove an instance from the instance dictionary by its class name.
        # Make sure to call ignoreAll() on all instances attached to this
        # object which inherit from DirectObject or else they won't be
        # deleted properly.
        if clsName in self.instances:
            instance = self.instances[clsName]
            if isinstance( instance, DirectObject ):
                instance.ignoreAll()
            del self.instances[clsName]
{% endhighlight %}

Note the two new methods, AttachScript() and DetachScript(). We will use these to add and remove additional code to our PandaObject. Code to be added will be its own class which I will refer to from here as a ‘behaviour’.

Behaviours are attached using the AttachScript method which takes a path to a python script as an argument. By using Python’s imp module we can import a module using a string, and it doesn’t have to be found on sys.path either. There is one caveat though – imp.find_module will only take a file name as its first argument, absolute or relative paths will not work. As I want this to handle relative paths I have to build a list of search paths to pass to find_module by joining the directory of the script to each of the sys.paths.

There is another restriction in that the name of the behaviour’s class must be named an uppercase leading match of the file that it lives in. While this goes against my ideas of dictating how the developer approaches their project, I don’t think it’s too restrictive and also matches the Python / Unity paradigm of naming classes in accordance to the files they live in.

Once we have found the class name we create an instance of it with the PandaObject’s node path as an argument. Any behaviour we add is probably going to modify the node path in some way, so we’ll need to pass this through. All instances are then stored in the PandaObject’s instances dictionary – something I’ll probably end up changing as currently it is not possible to attach more than one of the same type (named) of behaviour to a PandaObject.

We can remove instances that are attached to our PandaObject by passing the class name to DetachScript(). I’ve added a call to ignoreAll() for any behaviours that are inherited from DirectObject as they won’t be garbage collected without removing their reference from the messenger first.

Now to create some kind of basic behaviour. The following code will move the node path slowly up and down when started:

{% highlight python %}
import math
 
from direct.showbase.DirectObject import DirectObject
 
class NewBehaviour( DirectObject ):
 
    def __init__( self, np ):
 
        # Set the node path this behaviour will control then bind Start and
        # Stop events.
        self.np = np
        self.accept( 'StartNewBehaviour', self.Start )
        self.accept( 'StopNewBehaviour', self.Stop )
 
    def Update( self, task ):
 
        # This will move the node path up and down like a sine wave.
        self.np.setZ( self.initZ + math.cos( task.time ) )
 
        # Keep this task running so long as the node path is valid.
        if not self.np.isEmpty():
           return task.cont
 
    def Start( self ):
 
        # Get our initial z position then add an update task to be processed
        # every frame.
        self.initZ = self.np.getZ()
        self._task = taskMgr.add( self.Update, 'UpdateTask' )
 
    def Stop( self ):
 
        # Remove the update task from the task manager. Remember to set the
        # _task member to None or else the behaviour won't be garbage
        # collected!
        taskMgr.remove( self._task )
        self._task = None
{% endhighlight %}

Nothing too special going on here. We store the input node path and bind some events when the class is instantiated, and there’s some basic wrapping of the task manager going on there too.

Now to bring the whole thing together. We start by loading the default box model, parenting it under render and attaching a PandaObject. We then pull that object back out of the scene graph in a different scope and attach the behaviour to it by passing the file path to the script:

{% highlight python %}
import direct.directbase.DirectStart
 
from pandaObject import PandaObject
 
def Foo():
 
    # Load a model, put it in the scene graph and attach a PandaObject hook to
    # it
    box = loader.loadModel( 'box' )
    box.reparentTo( render )
    PandaObject( box )
 
def Bar():
 
    # Pull the node path out of the scene graph and attach the new behaviour
    # to it. myNp could be obtained other ways; returned from a collision
    # picking event for example.
    myNp = render.find( '*box*' )
    pObj = PandaObject.Get( myNp )
    pObj.AttachScript( 'newBehaviour.py' )
 
Foo()
Bar()
 
# Move the camera so we can see the box, then send the message to start the
# new behaviour.
base.cam.setY( -10 )
messenger.send( 'StartNewBehaviour' )
 
run()
{% endhighlight %}

Running the above code should show a box that floats up and down. To remove the node cleanly we have to make sure we stop the task running first, then we can remove it like normal:

{% highlight python %}
messenger.send( 'StopNewBehaviour' )
myNp = render.find( '*box*' )
PandaObject.Break( myNp )
myNp.removeNode()
{% endhighlight %}

So while this seems overly complex in order to get a box to move, you can imagine how neat this works when attached to a script file browser. By binding some drag and drop events we can easily attach the user’s script to any node path in the scene.