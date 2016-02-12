In the previous section we discussed the render tree, but only used static values for our node properties. All
of those properties can also take time-varying values to create rich animations. These values can come from
[user input](user-input.md) or `Transitionables`.

### Transitionables

[`Transitionables`](http://samsarajs.org/reference_docs/classes/Core.Transitionable.html) define a value or array 
of values that change over time. They are interpolated either by easing curves
or physics transitions, like springs. `Transitionables` like all Samsara [streams](streams.md) emit `start`, `update` and 
`end` events. To animate a value, call the `set` method with a target value, transition definition, and (optional) 
callback function. We'll discuss what we mean by a transition definition below, but first an example:

```js
var Transitionable = require('samsara/core/Transitionable');

var t = new Transitionable(0); // define a transitionable with initial value 0

t.on('start', function(value){
    console.log(value) // 0
});

t.on('update', function(value){
    console.log(value) // values between 0 and 100
});

t.on('end', function(value){
    console.log(value) // 100
});

t.set(
    100, // new value 
    {curve : 'easeIn', duration : 500},  // transition definition 
    function(){ console.log("finished"); // callback
});
```

Layout data of a node can read from `Transitionables`. For example, we can have a `Transitionable` fade out a `Surface`
with the following code:

```js
var opacity = new Transitionable(1);

var surface = new Surface({
    size : [100,100],
    properties : {background : 'red'},
    opacity : opacity
});

context.add(surface);

opacity.set(0, {curve : 'linear', duration : 500});
```

### Mapping values

Like all Samsara [streams](streams.md), `Transitionables` have a `map` method, which converts its values into other types. 
Calling `map` will return a new stream, so the same `transitionable` can map to several different streams.
Here we use a `Transitionable` to rotate a `Surface` and fade it out at the same time.

```js
var t = new Transitionable(0);

var rotation = t.map(function(angle){
    return Transform.rotateZ(2 * Math.PI * angle);
});

var opacity = t.map(function(x){
    return 1 - x / 2;
});

var surface = new Surface({
    content : 'click me',
    size : [100, 100],
    origin : [.5,.5],
    properties : {background : 'red'}
});

surface.on('click', function(){
    t.set(1, {curve : 'easeInOut', duration : 700});
});

var context = new Context();

context
    .add({        
        transform : rotation,        
        opacity : opacity,
        align : [.5,.5]
    })
    .add(surface);
```

<p data-height="266" data-theme-id="20796" data-slug-hash="JGZbOQ" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/JGZbOQ/'>transitionable-map</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

### Easing Curves

By default, `Transitionables` come with 9 predefined easing curves:

* `linear` (default)
* `easeIn`
* `easeOut`
* `easeInOut`
* `easeOutBounce`
* `easeInCubic`
* `easeOutCubic`
* `easeInOutCubic`
* `easeOutWall`

They are used with a transition definition, as in the examples above, which is a JSON object with the following properties:

| key | default |  description |
| --- | ------- |  ----------- |
| duration | 500 | The time in milliseconds to complete the transition |
| curve | 'linear' | The easing curve name |

#### Custom Easing Curves

Though Samsara only includes 9 easing curves by default, any easing curve can be added by providing your own easing function.
This function should be defined on the domain [0, 1] and map to the range [0,1]. You can map to values beyond the range [0,1]
which will  correspond to an undershoot (if less than 0) or overshoot (if greater than 1).

```js
var myCustomFunction = function(t){ return Math.pow(t, 5); }
var myTransitionable.set(100, {curve : myCustomFunction, duration : 500});
```

Or give your function a name and register it with the `Transitionable` constructor. You can then define it once
and use it from anywhere in your application.

```
Transitionable.registerCurve("myCustomCurve", myCustomFunction);
var myTransitionable.set(100, {curve : "myCustomCurve", duration : 500});
```

### Physics Curves

Physical curves can also be used to transition values. They lack the precise timing of easing curves,
but whereas easing curves come in a discrete number, physics curves are continuously parametrized. They can also
take into account the velocity of a previous motion, such as a gesture, to create more natural transitions. Currently,
the physics curves that Samsara comes with are springs and inertia.

The API for a physics curves is exactly like an easing curve except the transition definition is different.

##### Transition Definition for Springs

| key | default | Range | description |
| --- | ------- | ----- | ----------- |
| period | 100 | [0, ∞] | The time it takes for one oscillation without any damping applied |
| damping | 0.5 | [0, 1] | If 0 the spring will oscillate forever, if 1 the spring will never oscillate |
| velocity | 0 | [-∞, ∞] | The initial velocity of the transition in milliseconds/pixel |

For example:

```js
myTransitionable.set(1, {curve : 'spring', damping : .5, period : 100, velocity : -.1});
```

Here's the same example as above, but with a physics transition instead of an easing curve:

<p data-height="266" data-theme-id="20796" data-slug-hash="eJKByP" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/eJKByP/'>transitionable-map-physics</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

##### Transition Definition for Inertia

| key | default | Range | description |
| --- | ------- | ----- | ----------- |
| drag | 0.1 | [0, 1] | The ratio the velocity will decrease by each tick |
| velocity | 0 | [-∞, ∞] | The initial velocity of the transition in milliseconds/pixel |

For example:

```js
myTransitionable.set(1, {curve : 'inertia', drag : .5, velocity : -.1});
```
