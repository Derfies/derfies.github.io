---
layout: post
title:  "Attaching Scripts to Objects in Panda3D – Part I"
date:   2012-01-05
tags: [musings, panda3d, panda3d scene editor]
---
Now that I have a basic scene editor capable of adding, transforming and editing the properties of nodes, the next thing to do is to get some scripted behaviour happening. Having used Unity for my last project I’ve found my design choices to be greatly influenced by this tool, and Unity takes the intuitive approach of allowing the user to attach scripts to nodes in the scene hierarchy. Any subsequent manipulation of the node or its components by these scripts can be done using reserved keywords in the body of the script.

This idea of packaging code close to the node path it’s responsible for seems very object oriented and appeals greatly to me, and there’s no reason why we can’t do a similar thing in Panda3D. The most logical choice is to start subclassing NodePath or PandaNode, and adding the additional behaviour and methods there. Consider the following code:

{% highlight python %}
import direct.directbase.DirectStart
from panda3d.core import NodePath
 
class MyNodePath( NodePath ):
 
    def DoSomething( self ):
        print 'Hello world!'
 
def Foo():
 
    # Create an instance of our custom node path, stick it into the scene graph
    myNp = MyNodePath( 'myNodePath' )
    myNp.reparentTo( render )
    myNp.DoSomething()
 
def Bar():
 
    # We're in a different scope and have lost the original reference to myNp,
    # so we need to pull it out of the scene graph again
    myNp = render.find( 'myNodePath' )
    myNp.DoSomething()
 
Foo()
Bar()
 
run()
{% endhighlight %}

However when we run this we get:

{% highlight python %}
Hello world!
Traceback (most recent call last):
  File "test.py", line 24, in
    Bar()
  File "test.py", line 21, in Bar
    myNp.DoSomething()
AttributeError: 'libpanda.NodePath' object has no attribute 'DoSomething'
{% endhighlight %}

That’s odd – our method seems to have disappeared. Perhaps this is to be expected however, as a NodePath class is meant to serve as a path to a node – not a node itself. Maybe a reference to a node is just a wrapper generated during runtime. Let’s try a more concrete example by subclassing the PandaNode directly:

{% highlight python %}
import direct.directbase.DirectStart
from panda3d.core import PandaNode
 
class MyPandaNode( PandaNode ):
 
    def DoSomething( self ):
        print 'Hello world!'
 
def Foo():
 
    # Create an instance of our custom node
    myPNode = MyPandaNode( 'myPandaNode' )
    myPNode.DoSomething()
    render.attachNewNode( myPNode )
 
def Bar():
 
    # We're in a different scope and have lost the original reference to myNp,
    # so we need to pull it out of the scene graph again
    myNp = render.find( 'myPandaNode' )
    myNp.node().DoSomething()
 
Foo()
Bar()
 
run()
{% endhighlight %}

This doesn’t seem to work as expected either:

{% highlight python %}
Hello world!
Traceback (most recent call last):
  File "test.py", line 24, in
    Bar()
  File "test.py", line 21, in Bar
    myNp.node().DoSomething()
AttributeError: 'libpanda.PandaNode' object has no attribute 'DoSomething'
{% endhighlight %}

So what the Sam Hill is going on here?  I created an instance of MyPandaNode, stuck it in the scene graph but can’t access its method once I’ve pulled it back out again. This seems like an odd limitation and one that on searching the Panda3D forums seems to bite everyone sooner or later. It happens because Panda is essentially a C++ engine under the hood, and whenever you perform a query that returns a PandaNode or NodePath you are handed the C++ object with a thin Python wrapper around it. Unless you keep track of your custom NodePaths and never lose references to them you’ll always get a newly minted, default Python wrapper returned. Subclassing in Python merely subclasses the wrapper class, not the C++ class.

This makes our quest to attach code to object a lot more difficult, but there should still be a straightforward solution. If we can’t subclass NodePath, then surely we should be able to wrap it. This works fine but we need to get back our custom class once we have the NodePath. Thankfully NodePath has a great little method which can be used to attach a python object to a NodePath, this will stay attached even if we lose the initial reference to the NodePath and have to pull it out of the scene graph again. It’s called ‘setPythonTag’ and can be used to back reference the NodePath to your custom class:

{% highlight python %}
class MyObject( object ):
 
    def __init__( self, name ):
        self.np = NodePath( name )
        self.np.setPythonTag( 'base', self )
{% endhighlight %}

Now whenever we have a NodePath we can simply get back to our custom class by calling:

{% highlight python %}
myNp.getPythonTag( 'base' )
{% endhighlight %}

Neat – we’ve essentially solved our problem! Yes, but we’re also introduced a new one. Consider the following:

{% highlight python %}
import direct.directbase.DirectStart
from panda3d.core import NodePath
 
class MyObject( object ):
 
    def __init__( self, name ):
        self.np = NodePath( name )
        self.np.setPythonTag( 'base', self )
 
    def __del__( self ):
        print 'deleted!'
 
    def DoSomething( self ):
        print 'Hello world!'
 
def Foo():
 
    # Create an instance of our custom class, stick the node path into the scene
    # graph
    myObj = MyObject( 'myNodePath' )
    myObj.np.reparentTo( render )
    myObj.DoSomething()
 
def Bar():
 
    # We're in a different scope and have lost the original reference to myNp,
    # so we need to pull it out of the scene graph again
    myNp = render.find( 'myNodePath' )
    myNp.getPythonTag( 'base' ).DoSomething()
 
    # Try to remove the node from the scene graph, we should see the deleted
    # message from the destructor
    #myNp.clearPythonTag( 'base' )      # Uncommenting this line will result in the node path being destroyed properly
    myNp.removeNode()
 
Foo()
Bar()
 
run()
{% endhighlight %}

You can see from running the above script that you will never see the ‘deleted!’ message printed from the destructor, unless the python tag is cleared first. This is because essentially we’ve created a circular reference: The class contains a reference to the NodePath, which contains a reference back to the class as a Python tag. While this isn’t a huge problem if we remember to break the reference, it can cause memory leaks if not dealt with properly. If you detach a NodePath from the scene graph expecting it to be deleted and lose track of your instance of MyObject the two will never be garbage collected – and neither will you be able to access either of them to break the reference.

### Can Weakref Help?
In short: no. At least, not obviously. I made the mistake of thinking that the problem could be solved by making the back reference a weak reference, which meant that Python’s garbage collection would properly destroy the NodePath once all other references had been removed. For example:

{% highlight python %}
self.np.setPythonTag( 'base', weakref.proxy( self ) )
{% endhighlight %}

Unfortuantely you need to have at least one non-weak reference to an object or it will be removed as soon as you drop out of the scope in which it was created, leaving us back with the intial problem. Using weakref doesn’t look like it will be a magic bullet, you would have to use it in conjunction with another solution…

### Other Potential Solutions
It would be possible to keep track of each node path’s custom object using a dictionary as part of a manager class, but I think you would eventually come up against the same problem. In order to remove a node path from the scene graph properly you would need to make sure you removed it from the dictionary in the manager object as well.

I considered other wacky solutions like storing values and functions as individual python tags on a node path, but you would need a manager class which knew how to access and run the code. In short, I don’t think it would be a very elegant solution.

### So What is the Solution?
At the moment I’m leaning more to the circular reference idea, and adding an additional Break() method to break the circle and allow garbage to be collected normally. I’ve also opted to go with a static method to retrieve the PandaObject from the NodePath, as this keeps things nice and neat; another developer doesn’t have to know the name of the tag this class is hidden in either, so it looks rather clean:

{% highlight python %}
class PandaObject( object ):
 
    def __init__( self, np ):
 
        # Store the node path with a reference to this class attached to it
        self.np = np
        self.np.setPythonTag( 'PandaObject', self )
 
    @staticmethod
    def Get( np ):
 
        # Return the panda object for the supplied node path
        return np.getPythonTag( 'PandaObject' )
 
    @staticmethod
    def Break( np ):
 
        # Clear the panda object tag to allow for proper garbage collection
        np.clearPythonTag( 'PandaObject' )
 
    def DoSomething( self ):
        print 'Hello world!'
{% endhighlight %}

This means calling custom methods on our NodePath will look something like this:

{% highlight python %}
pObj = PandaObject.Get( render.find( 'myNodePath' ) )
pObj.DoSomething()
{% endhighlight %}

If we want to completely remove a node, we have to remember to call Break() before detaching it to make sure the circular reference is broken.

In summary, setPythonTag() is your friend. Just make sure to break any circular references you create or otherwise your NodePaths won’t be collected as you might expect. In my next post I’ll show some other creative uses for the technique described above.