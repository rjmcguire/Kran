Kran
====

A flexible, convenient, simple and efficient architecture primarily for game
development. It will get your game development up to speed quickly and allow
your game to expand smoothly.

Kran provides nothing but an _architecture_ that can be used for any type of
game. All other decisions - like how to do rendering or physics - is up to the
user. Kran can without problems be used in conjunction with other libraries.

_Note_: Kran is under heavy development, everything is work in progress.

# cljan
If you're interested in Clojure and/or functional programming take a look at
[VincentToups](https://github.com/VincentToups)' awesome
[Cljan](https://github.com/VincentToups/cljan). A game-engine independent,
purely functional entity component system inspired by Kran.

Demo game
=========
[Try the crazy demo game with explosions and blood!]
(http://paldepind.github.io/Kran/examples/circlebomb/)

_Note_: It only works in modern desktop browsers - currently only tested in Firefox and
Chrome.
![Screenshot](http://i.imgur.com/H9Qijoc.png)

Why Kran?
=========

It's _flexible_:
Kran gains most of its flexibility from the entity system architecture which it is
an implementation of. Unlike the case with object oriented programming data
isn't tied together with the functions that processes it. And you don't have
to fit things inside an inflexible class hierarchy to share code. In Kran
entities are created by composition rather than inheritance. They can be
composed out of any combination of data components desirable and they'll
automatically find their way to the functions that wants to operate on them.
This allows for maximum flexibility and reuseability - nothing is tightly
coupled, everything can be mixed to ones hearts desire.

It's _effortless_:
Kran has a simple convenient API that makes common tasks very easy. It provides
structure to your code so that you can worry about actually getting your game
rolling without being hindered later on by inflexibility or a cumbersome design
that makes further expansions hard. It has an event system that integrates with
DOM events and makes using the pub/sub pattern straightforward. It shows you
the way without getting in you way.

It's _simple_:
We aren't kidding when we claim that Kran is a tiny library. We makes other
so called tiny libraries look gigantic. Kran gives you an architecture and
that's all. It has a tight focus. This means you can learn it quickly and
easily get a complete understanding of how it works. It won't impact your page
loading time in any noticeable way either.

It's _efficient_: Kran is designed from the bottom up with an efficient use of
data structures and an API that doesn't introduce garbage during game runtime.
Note however that Kran is in an early stage of development and the optimization
isn't done.

Crash course
============

In an entity system all data/state is stored inside _components_. Components
are tiny bits of tightly related data. It's a property/characteristic that
a thing can possess. That could be a position component, a shape component
or a color component. In Kran a component can be defined by passing a
constructor function to `kran.component` where `kran` is an instance of `Kran`:

```javascript
var kran = new Kran()

var velocity = kran.component(function (x, y) {
  this.x = x || 0
  this.y = y || 0
}
```

Components with just one property can be created using the shorthand syntax
shown below
```javascript
var weight = kran.component("val")
```
This is the same as doing
```javascript
var weight = kran.component(function (weight) {
  this.val = weight
})
```

We've now created two components. A velocity component and a weight component.
We want to make all entities that has both speed and a weight to accelerated
towards the ground. In other words we want to implement gravity. For that we need
a _system_. Systems is were the logic that would be methods in OOP is put in an
entity system.

```javascript
kran.system({ // Creates a new system that accelerates objects with weight towards the ground
  // The system is interested in entities with velocity and weight components
  components: [velocity, weight],
  // When the system runs this function will be called for each entity that has the
  // requested components, the components will be given as arguments to the function
  every: function(velocity, weight) {
    velocity.y += weight.val * GRAVITY
  }
})
```

That's all we have to do in order to make entities get attracted towards the
ground! The system is automatically registered inside Kran, and now when
systems are being run in the main loop the above system will automatically be
run for every entity that satisfies it's dependencies.

The final thing we need to do is compose some entities that the system can
operate on. Fortunately it's super easy! `Kran.entity` creates a new entity and
returns it. The returned entity has an `add` method that as its first argument
takes the component to add. Further arguments will be passed along to the
components constructor function. It returns the entity to allow for chaining.

```javascript
// Creates a new entity, adds two components to it and initializes them
kran.entity().add(velocity, 100, 0).add(weight, 10)
// This component is heavy and heading right towards the ground, might be a meteor ;)
kran.entity().add(velocity, 0, -10000).add(weight, 84782)
```

Now we set up the game loop. This is what a simple game loop might look like in
Kran (actually this is exactly what the demo game's loop look like):
```javascript
var gameLoop = function() {      
  kran.run("all")
  requestAnimationFrame(gameLoop)
}  
```
The above simply runs all systems in order of creation. If more control is needed
systems can be put into groups and ran like this:
```javascript
kran.run("group-name")
```
That's all for now. Kran is still in heavy development and more documentation
will be created in the future. As of now take a look at the 
[source code of the example game](https://github.com/paldepind/Kran/blob/master/examples/circlebomb/game.js)
for an example on how a game made with Kran can look like.

API documentation
=============
_Note:_ The API documentation is still very rough and should be considered a
draft.

### The Kran object
A Kran "universe" is created by instantiating the Kran object.
```javascript
var kran = new Kran()
```
This "univers" contains a seperate collection of components, systems and
entities.

### Creating components
Components are created with the `component` function on a Kran instance. The
function supports three different ways of creating components. Either by
passing it a constructor function, a string or undefined.

The basic way of creating components is by passing `component` a constructor
function.
```javascript
kran.component(constr)
```

Here `constr` is a constructor function that initializes the component. Since
the constructor function is a normal JavaScript constructor function it is
perfectly possible to add methods to component. That however is not proper
usage of a entity system and should probably never be done. The constructor
function gets passed the argument given when an instance of the component is
added to an entity.

_Example:_
```javascript
var pos = kran.component(function (x, y) {
  this.x = x
  this.y = y
})
```

For creating a component with only one property `component` provides a
convenient shorthand syntax. Simply call it with the name of the single
component as a string.
```javascript
kran.component(propertyName)
```
Here `propertyName` is a string and the above is identical to.
```javascript
kran.component(function (property) {
  this.propertyName = property
})
```

If one needs a component where only its presence if of concern (i.e. it doesn't
contain any changing data) `component` can be called with no arguments
```javascript
kran.component()
```
This creates a component that takes no initialization arguments and contains no
properties. Kran will store it as a boolean internally.

### Registering systems

Sysems are registered by calling `system` method with an object describing the
system.
```javascript
kran.system(systemObj)
```

A `systemObj` is an object and can contain the following properties

* `components`: A component id or an array of component ids that the function
depends on.
* `pre`: A function called once before the `every` function when the system is
run.
* `every`: A function being called once for every entity that has the
component required by the component. As arguments the function is given the
entities components followed by the entity itself.
* `post`: Like `every` except it is called after `every`.
* `arrival`: Called whenever an entity not satisfying the system gets a
  component added making it satisfy the system. The components and the entity
  is passed as arguments.

### Running systems
When systems groups are created a function with the same name as the group will
be attached to `kran.system`. It allows running all the systems in the group
in order of creation.
```javascript
kran.run("group-name")
```

Alternatively all systems except those listening to global events gets added to
a special group named `all`. Thus all systems can be run with
```javascript
kran.run("all")
```

### Creating entities
A new entity is created by calling the `entity` method on a Kran instance. The
function takes no arguments. It returns an entity object.

*Example:*
```javascript
var ent = kran.entity()
```
`ent` is now an entity object with no components. I.e. it's an empty container.

### Composing entities
Composing entities is easy and consice. All the entity methods returns themselves
to allow for chaining.
*Example:*
```javascript
ent.add(fleeing, speed).remove(elastic).add(firing, interval, damage)
```

### Adding components
Components are added with the `Entity#add` method. The first argument to which
is a component followed by a variable amount of arguments that will be passed
along to the components constructor function if any.
*Example:*
```javascript
entity().add(componentId, arg1, arg2, arg3)
```

### Removing components
Components are removed with the `Entity#remove` method.
*Example:*
```javascript
hero.remove(awesomePerk)
```

### Triggering components
The `Entity#trigger` method is a convenience method for adding a component
and immediatly removing it again.

This
```javascript
tree.trigger(leavesDropped, 4)
```
is equivalent to this:
```javascript
tree.add(leavesDropped, 4).remove(leavesDropped)
```

The former is more consice and more clearly communicates the intention of
the addition and removal of the component. With this simple function and the
`arrival` hook in systems Kran implements gets and integrated event system
for free.

### Working with entity collections
Kran has a concept of entity collections. These are collections of entities
containing a given set of components. Kran will update these collections when
needed as components are added to and removed from entities. Internally these
entity collections are used by systems but they have other uses and thus they
are also exposed in the API.

*Example:*
```javascript
kran.getEntityCollection([comp1, comp2, comp3])
```

### Linked list

```javascript
new kran.LinkedList()
```

```javascript
linkedListInstance.add(data)
```

To do
=====

* Improve documentation
* Add benchmark suite
* Add object pooling for internal and external usage
* More example games
* Possible extensions
  * JSON serialization and restoration of all state for save game
  * Socket.IO integration (highly speculative)
