A [**`View`**](http://samsarajs.org/reference_docs/classes/Core.View.html) is a way to encapsulate a part of the render 
tree. `Views` can be added to the render tree
just like nodes and `Surfaces`. They can also build their internal render tree with their `add` method as [previously
mentioned](render-tree.md#views). Moreover, they have built-in support for events, options with defaults, 
and manage their own size, origin and opacity.

### Custom Views

Samsara includes a [`View` constructor class](http://samsarajs.org/reference_docs/classes/Core.View.html) which is used to make custom `Views`.
It has an `extend` method which takes a JSON object of custom properties and methods. 
A few keys are reserved for special purposes. They are:

| key | description |
| --- | ----------- |
| `defaults` | A JSON object of default options. When a `View` is instantiated, any options that are not specified are replaced by their default.|
| `events` | A JSON object of input events, with keys being the name of the event and values being either a function to handle the event, or a string that matches an instance method to handle the event.|
| `initialize` | A function that is called when the `View` is instantiated. |

Here's an example of creating a custom `View`:

```js
var MyView = View.extend({
    defaults : {
        content : '',
        width : 42
    },
    events : {
        hello : "onHello"
    },
    // Called on instantiation
    initialize : function(options){		   
        // Options are passed in after being patched by the defaults
        // They are also stored in this.options

        var surface = new Surface({
            origin : [.5,.5],
            content : options.content,
            size : [options.width, 100],
            properties : {background : 'red'}
        });

        this.add({align : [.5,.5}).add(surface);
    },
    onHello : function(data){
        console(data.msg);
    }
});
```

This view has default options and listens to a `hello` event. We can instantiate an instance of this `View` and
override the default options (if necessary) like so

```js
var myView = new MyView({
    content : 'hello'
});
```

We can send `myView` a `hello` event to execute its `onHello` method:

```js
myView.trigger('hello', {msg : 'hello'}); // hello
```

Or also call the `onHello` method direction with `myView.onHello()`. And lastly, `Views` can be added to the render
tree just like nodes and `Surfaces`.

```js
context.add(myView);
```

### Render Tree Methods

`Views` have an `add` method for building up their own internal render trees. They accept nodes, `Surfaces`
or other `Views`. Here's an example of a `View` that encapsulates a cross-fade between two `Surfaces`.

```js
var MyView = View.extend({
    initialize : function(){
    	this.opacity1 = new Transitionable(0);
		
    	var opacity2 = this.opacity1.map(function(value){
       		return 1 - value;
    	});
		
    	var surface1 = new Surface({
     	    opacity : this.opacity1,
    	    properties : {background : 'red'}
    	});
		
    	var surface2 = new Surface({
    	    opacity : opacity2,
    	    properties : {background : 'blue'}
    	});
		
    	this.add(surface1);
    	this.add(surface2);
    },
    setCrossfade : function(value, transition, callback){
    	this.opacity1.set(value, transition, callback);
    } 
});
```

<p data-height="266" data-theme-id="20796" data-slug-hash="PZamwZ" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/PZamwZ/'>View Crossfade</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

### Event Methods

`Views` can broadcast and receive events. A `View` has two `EventHandlers`, one for input, and
one for output.

#### Input

All `Views` have a `input` property that which listens to incoming events. `Views` can subscribe
to events, or have events triggered upon them.

```js
var MyView = View.extend({
    initialize : function(){
        this.input.on('foo', function(data){
            console.log(data.msg);
        });
    }
});

var emitter = new EventEmitter();

myView.subscribe(emitter);
emitter.emit('foo', {msg : 'bar'});

// or

myView.trigger('foo', {msg : 'bar'});
```

When extending a `View`, the `events` dictionary is a shorthand for adding listeners to the `input`
property (and auto-binds `this`).

#### Output

`Views` also have an `emit` method for emitting events.

```js
var MyView = View.extend({
    sendFoo : function(){
        this.emit('foo');
    }
});

var myView = new MyView();
myView.sendFoo();
```

A `View's` `emit` method is actually shorthand for `view.output.emit`. The `output` property, like a
`View's` `input` property, is also an `EventHandler`. 

#### Views as Streams

As we've mentioned, events inbound to the `View` are received by the `input`, and events outbound from the `View` go through the `output`.
This is particularly useful for thinking of `View's` themselves as streams. A `View` can
listen on a `Transitionable`, or a Samsara `input`, and can output a stream as well.

```js
var MyView = View.extend({
    initialize : function(){
    	var processedInput = this.input.map(function(value){
    	    return 1 + value;
    	});
    	
        this.output.subscribe(processedInput);
    }
});

var myView = new MyView();

var t = new Transitionable(0);

//myView.input is now getting events from the transitionable
myView.subscribe(t);

myView.on('start', function(value){
    console.log(value) // 1
});

//myView.output is emitting the processed events
myView.on('end', function(value){
    console.log(value) // 2
});
```

For example, in the [SideMenu demo](https://github.com/dmvaldman/samsara/tree/master/examples/SideMenu) the blue
`Content` View is getting input from the drag gesture, clamping and normalizing it, and sending that
value to the `Drawer` and `Nav`.

### Size, Origin and Opacity Methods

`Views`, like `Surfaces` can take size, origin and opacity properties on instantiation.

```js
var myView = new MyView({
    origin : [.5,.5],
    proportions : [.5, .5]
});

context.add({align : [.5,.5]}.add(myView);
```

Here's an example of instantiating the cross-fade view from before with these size options:

<p data-height="266" data-theme-id="20796" data-slug-hash="mVKmJy" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/mVKmJy/'>View Crossfade Sized</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

`Views`, like `Surfaces` also have the same setters for size, origin and opacity:

* `setSize`
* `setProportions`
* `setMargins`
* `setAspectRatio`
* `setOrigin`
* `setOpacity`

### Size Stream

`Views`, like `Surfaces`, have a `size` stream property. In the following example, we render
two `Surfaces` one on top the other if the `View's` height is larger than its width, and side
by side if the `View's` height is smaller than its width.

```js
var MyView = View.extend({
	initialize : function(){
	    var proportions1 = this.size.map(function(size){
	    	return (size[1] > size[0])
	    	    ? [1, 1/2]
	    	    : [1/2, 1];
	    });
	    
	    var proportions2 = this.size.map(function(size){
		    return (size[1] > size[0])
		        ? [1, 1/2]
		        : [1/2, 1];
		});
	    
        var surface1 = new Surface({
            proportions : proportions1,
            properties : {background : 'red'}
        });
		
        var surface2 = new Surface({
            proportions : proportions2,
            properties : {background : 'blue'}
        });
		
        var displacement2 = this.size.map(function(size){
            return (size[1] > size[0])
               ? Transform.translateY(size[1]/2)
               : Transform.translateX(size[0]/2);
        });
		
        this.add(surface1);
        this.add({transform : displacement2}).add(surface2);
    }
});
```

Here's a demo, but you'll need to view the demo on Codepen and resize it to see it in action.

<p data-height="266" data-theme-id="20796" data-slug-hash="xZzqoy" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/xZzqoy/'>View Size</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>