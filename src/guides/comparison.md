### MVC Frameworks

MVC frameworks, be it React, Angular, Ember, Backbone, Vue, etc are fundamentally
about handling the [business logic](https://en.wikipedia.org/wiki/Business_logic) 
of an app. This is a large part of development, but not the only part.
The difference is between [content and presentation](https://en.wikipedia.org/wiki/Theory_of_Forms). 
These frameworks are primarily concerned with the content of an application. Conversely, Samsara
only moves rectangles around the screen, and is indifferent to what happens inside them. 
Samsara doesn’t include any support for routing, server syncing, templating, data-binding
and other features of most MVC frameworks. In this way, Samsara is more closely comparable to CSS's 
flex-box than it is an MVC library. 

As such it is not a complete package for application development. Samsara is meant to work with other 
tools as part of the developer workflow. A `Surface` is meant to be populated 
with content from a React component, or a Handlebars template, or provided to a Backbone view, etc. 
All that is needed is to associate a `Surface` with any of these content sources, and call `surface.setContent()` 
when the content changes. That said Samsara is very opinionated about the presentation layer. 
If Samsara doesn't integrate well with your library of choice. [Please let us know](https://groups.google.com/forum/#!forum/samsarajs) 
and we will strive to improve its compatibility. We will soon be working on integrations with React, Backbone and Vue.

##### Shortcomings of MVC

MVC frameworks by their very nature have two other shortcomings when it comes to the presentation layer:

* _Representing view logic in a model is an antipattern_

The _V_ in _MVC_ should not care about where it is in space; this is the responsibility of something
outside the view. It doesn't follow to have `this.props.xPosition = 0` in a React class, or `{x : 0}`
in a Backbone model, etc. A view should exist in a platonic universe and be explicitly brought
down to reality by giving it physical properties like size, and position. 
This is the separation of concerns between the business logic, and the presentation logic.

* _Data-binding for data that changes at 60 FPS is different_

The concerns about data that changes at 60 FPS are not the same as the concerns about data that
changes every now and again. For instance, using React's diffing strategy is too costly for
complex and performant animations. The paradigms of getters, setters and state from the MVC world is not as
expressive as streams, subscriptions and eventing. MVC is a paradigm for data that changes discretely. For
continuously changing data, functional reactive programming is better suited.

### FRP Frameworks

[Functional reactive programming](https://en.wikipedia.org/wiki/Functional_reactive_programming) (FRP) 
is another approach to application design. Instead of getters, setters and state, FRP favors notions of continuous 
dataflow and events. In FRP, instead of state being something that is owned by objects
(in an OOP sense), state is something that flows through an application, with objects acting more like functions that 
transform the state, without holding onto it. This gets around a lot of the programming problems of synchronizing state
and side-effect bugs.

Samsara borrows heavily from FRP principles, as do other recent JavaScript frameworks such as Elm and Cycle. The
difference between Samsara and these libraries is less one of philosophy, and more one of application. Elm
and Cycle are approaches to handling all application state in an FRP way, while Samsara is only concerned with 
the state of the presentation.

For instance, a basic example from both 
[Cycle](http://cycle.js.org/basic-examples.html#increment-and-decrement-a-counter) and 
[Elm](https://github.com/evancz/elm-architecture-tutorial#example-1-a-counter) 
is incrementing a counter. Samsara would never be concerned with this use case for the simple reason that a counter doesn't 
animate at 60 FPS. A counter is part of the business logic, not the presentation.

##### Shortcomings of FRP

I personally believe that FRP approaches are better than OOP approaches for specific use cases, and worse
for others. By thinking of state as a flow through an application, FRP is great for managing the
parts of an application than can be seen as a cascading of information: where
one part of an application receives some data, transforms it, and hands it off to another part. In these cases
no part of an application needs to hold on to state. The problem arises when a developer doesn't know when
two parts of an application will interact. In these cases it's important for state to be held on to. A good
example would be collisions in a game: you don't know when a bullet will hit a player, for instance, but when it does, 
the collision needs to have access to the bullet's speed, and the player's bounding box, etc. This information
is better stored in objects until the time it is needed.
 
For this reason I don't think that FRP is the ultimate answer to application development, but it can be the answer
for a piece. Samsara is about thinking of layout as a cascade of information that begins with some continuous flow of 
user input and ends in the continuous flow of layout, which falls into the use case of FRP I think it is most suited for.

### Animation Frameworks

There are many libraries for doing animation on the web: Velocity.js, GreenSock, Bounce.js, etc. These libraries allow the
animation of a DOM element with a simple API. Perhaps something that looks like this:
                                                                                                 
```js
SomeLibrary.animate('#myId', {
    opacity : 1,
    duration : 1000,
    curve : 'easeIn'
});
```
 
Where Samsara is different is that it enables the coordinate hundreds of animations on hundreds of DOM nodes in a highly
dependent environment: where the layout and size of one node may depend on the layouts and sizes of others. 
The goal is to make the developer feel as if she is the conductor of an orchestra, not the player
of an instrument. It is the coordination of many animations that is the hard part. It is also the part that native 
iOS and Android apps have, and the web lacks.