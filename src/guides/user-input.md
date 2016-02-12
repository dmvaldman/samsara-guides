Samsara interprets continuous user input as a stream. Lest we forget, user input has always been a stream. When we write,

```js
window.addEventListener('mousemove', function(event){
    console.log(event.offsetX);
});
```

we are subscribing to a stream of data emitted from the DOM. But the DOM only starts us off: it gives us a spout to listen to.
The power of streams comes from their ability to transform and stitch together to build pipelines. As it stands with the DOM, 
the only way to process its streaming input data is to put all your business logic into the listener itself, which is a boon
to modularity and flexibility.

### Input Streams

Samsara gets around these problems by converting these DOM events into its own [streams](streams.md). Just like a `Transitionable`, values from user input
can be subscribed from and mapped directly to nodes to affect layout. Samsara comes with the following input streams:

| Input | Description | Data |
| ----- | ----------- | ------- |
| [MouseInput](http://samsarajs.org/reference_docs/classes/Inputs.MouseInput.html) | Captures mouse drag events | `delta`, `value`, `cumulate`, `velocity`, `clientX`, `clientY`, `offsetX`, `offsetY`|
| [TouchInput](http://samsarajs.org/reference_docs/classes/Inputs.TouchInput.html) | Captures touch events | `delta`, `value`, `cumulate`, `velocity`, `clientX`, `clientY`, `count`, `touchId` |
| [ScrollInput](http://samsarajs.org/reference_docs/classes/Inputs.TouchInput.html) | Captures mouse wheel and trackpad events | `delta`, `value`, `cumulate`, `velocity`, `clientX`, `clientY`, `offsetX`, `offsetY`|
| [PinchInput](http://samsarajs.org/reference_docs/classes/Inputs.PinchInput.html) | Captures the distance between two finger events | `delta`, `value`, `velocity`, `center`, `touchIds` |
| [RotateInput](http://samsarajs.org/reference_docs/classes/Inputs.RotateInput.html) | Captures the rotation between two finger events | `delta`, `value`, `velocity`, `center`, `touchIds` |
| [ScaleInput](http://samsarajs.org/reference_docs/classes/Inputs.ScaleInput.html) | Captures the relative stretching between two finger events| `delta`, `scale`, `distance`, `center`, `touchIds` |

All `Samsara` streams emit `start`, `update` and `end` events. For input events, these are derived from the native DOM events. 
For instance, `MouseInput` listens to mouse drag events, and will internally convert `mousedown`, `mousemove` and `mouseup` into
`start`, `update` and `end` respectively. Similarly, `TouchInput` will convert `touchstart`, `touchmove` and `touchend` into
`start`, `update` and `end`. `Scrollinput` will define an `end` event by debouncing DOM `wheel` events (since there
is no DOM `wheelend` event).

Samsara inputs do more than just unify events into a common syntax. They also provide derived data like `deltas`, 
`velocities`, and other quantities that it computes internally from the raw DOM event information. This makes it easy
to recognize gestures, apply effects like carrying the velocity of a drag into another transition, and combine various
sources of user input together. Here's an example of some of the data provided by `MouseInput`:

<p data-height="266" data-theme-id="20796" data-slug-hash="PZabwZ" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/PZabwZ/'>input-data</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>
<br>

Inputs need to subscribe to a source of DOM event data, such as a `Context` or `Surface`. They will then be listening to
DOM events originating from the source. The following code indicates how to have a `Surface` follow a mouse's position.

```js
var mouse = new MouseInput();

// The mouse now listens to the DOM events originating from the surface
mouse.subscribe(surface);

// Map the mouse date to a translation
var transform = mouse.map(function(data){
    return Transform.translate(data.cumulate);
});
```

<p data-height="266" data-theme-id="20796" data-slug-hash="OMERNY" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/OMERNY/'>mouse-drag</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

### Unifying Inputs with GenericInput

Samsara comes with one other input, called [`GenericInput`](http://samsarajs.org/reference_docs/classes/Inputs.GenericInput.html), 
which acts to unify several inputs together. This is useful for unifying
desktop and mobile experiences. For instance, you can unify touch and mouse events into one input, and this will behave
appropriately on desktop environments and touch screens. Another example is unifying scroll and touch events, to get consistent behavior
between a mouse/trackpad and a touch interface. 

You can register various Samsara inputs (similar to [registering custom easing curves](animation.md#custom-easing-curves) 
in a `Transitionable`) for `GenericSync` globally, and then later instantiate specific subsets of them where you need. 
Here's an example of how to use `GenericInput`.

```
var MouseInput = require('samsara/inputs/MouseInput');
var TouchInput = require('samsara/inputs/TouchInput');
var ScrollInput = require('samsara/inputs/ScrollInput');
var GenericInput = require('samsara/inputs/GenericInput');

// in main.js
// register all the inputs you need in {id : constructor} pairs
// the Ids registered can be used globally in the application
GenericInput.register({
   mouse : MouseInput,
   touch : TouchInput,
   scroll : ScrollInput
});

// in myFile.js
// create an instance of an input that unifies some subset of the registered input Ids
var mouseTouchInput = new GenericInput(['mouse', 'touch']);
var scrollInput = new GenericInput(['scroll', 'touch']);
```