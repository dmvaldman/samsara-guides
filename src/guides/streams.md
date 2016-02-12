Thus far we've been hearing about the concept of streams in terms of [Transitionables](animation.md) and [Samsara inputs](user-input.md). 
We have yet to precisely describe what we mean by a stream. In Samsara, 

* A [**stream**](http://samsarajs.org/reference_docs/classes/Streams.Stream.html) is an `EventEmitter` that emits 
`start`, `update` and `end` events, where where zero or more `update` events come between a `start` and an `end` event.

That's it really. But when taken from this abstract viewpoint streams are a powerful concept to describe animation, and more generally
anything that changes over time. For example, the entire render tree is also a stream: the request animation frame loop
that is Samsara's internal clock is canceled when the render tree emits an `end` event, and starts again when
the render tree emits a `start` event. This ensures that when nothing is changing, no JavaScript is being executed.

Though this opinion on what events need to be emitted to turn your ordinary `EventEmitter` into a stream is the only
necessary requirement, streams are better described by "stream logic": how to plug them together to build pipelines, 
how to transform them, how to combine and split them. For that reason, Samsara streams have methods to accomplish
these tasks.
 
### Transforming Streams

Transforming streams is the process of converting the data of one stream, into another stream with different data. There
are several common [methods](http://samsarajs.org/reference_docs/classes/Streams.Stream.html#methods) for doing this:

##### .map(`<Function>`) 

The `map` method transforms a stream of data into a new stream of mapped data.

```js
var t = new Transitionable(0);
var transform = t.map(function(value){
    return Transform.translateX(value);
});
```

##### .pluck(`<String>`)

The `pluck` method returns a piece of data from a stream, selected by key.

```js
// example of selecting a value of a JSON object
var mouse = new MouseInput();
var delta = mouse.pluck('delta');
```

If a stream returns an array, you can also pluck by the array index (for JavaScript array[0] and array['0'] are
the same anyway).

```js
// example of selecting an index in an array
var t = new Transitionable([0,0]);
var x = t.pluck(0);
var y = t.pluck(1);
```

##### .filter(`<Function>`)

The `filter` method takes a function which returns a boolean, and creates a new stream that only emits events if the filter 
function is satisfied (returns `true`).

```js
var t = new Transitionable(0);
var s = t.filter(function(value){
    return value < 0.5;
});
```

## Combining Streams

The methods above are for transforming a single type of stream into another, but sometimes you want to make a stream
from multiple streams. For combining streams there are two methods on the `Stream` constructor.

##### Stream.lift(`<Function>`, `<Array>`)

The `lift` method takes a function and an array of streams as arguments and produces a new stream whose values are
the return values of the supplied function. The current value of each of the streams in the
array is used as an argument into the function.
 
```js
var s = new Transitionable(0);
var t = new Transitionable(1);

// this stream will return the sum of the values of `s` and `t`
var liftedStream = Stream.lift(function(s, t){
    return s + t;
}, [s, t]);
```

##### Stream.merge(`<Object>`)

The `merge` method combines multiple stream sources into one. It takes a JSON object of streams (or static values) 
and creates a stream from them whose values are the current values of each of the streams. In fact, every node
in Samsara is simply a merged stream.

```js
var s = new Transitionable(0);
var t = new Transitionable(1);

// this stream will return an object consisting of the current values of `s`, `t` and `a` (which doesn't change)
var mergedStream = Stream.merge({
    s : s,
    t : t,
    a : [1,2,3]
});
```

##### Batching Events

Both the `lift` and `merge` methods need to handle the case when two or more of their sources are changing simultaneously.
Internally, events will be batched by the request animation frame loop. In the above `merge` example, if both `s` and `t` emit an `update` event in the same
request animation frame loop. For example, the `mergedStream` will only emit a single `update` event, with the most recent values
of both `s` and `t`.

##### Event Algebra (Mathematical Interlude)

Batching when events are not of the same type gets more interesting. To ensure consistency, one needs
to define an "event algebra". 

* if one source emits an `update`, the combined stream emits an `update`
* if all sources emit `start`, the combined stream emits `start`
* if all sources emit `end`, the combined stream emits `end`
* if some sources emit `start` and the rest emit `end` the combined stream emits `update` event.

Here are all the cases:

| source event | source event | combined event |
| ------------ | ------------ | ------------ |
| `start` | `start` | `start` |
| `start` | `update` | `update` |
| `start` | `end` | `update` |
| `update` | `update` | `update` |
| `update` | `end` | `update` |
| `end` | `end` | `end` |

If we begin with the premise that any stream that that emits `start` must eventually emit an `end` event. It is only with
these combiner rules that we can guarantee that every combined stream that emits a `start` event also emits an `end`
event. Because of the particulars of how batching occurs in Samsara, we couldn't use an established stream library like Bacon.js or Rx.js.

## Built-in Streams

Besides `Transitionables` and `inputs`, Samsara comes with several other streams that are particularly useful.

#### Accumulator

An [**`Accumulator`**](http://samsarajs.org/reference_docs/classes/Streams.Accumulator.html) is a stream that sums the values it subscribes from. This is often useful
for summing up deltas from other streams to get a total value.

```js
var accumulator = new Accumulator(0);
var mouse = new MouseInput();
accumulator.subscribe(mouse.pluck('delta'));

accumulator.on('update', function(value){
    console.log(value);
});
```

#### Differential

A [**`Differential`**](http://samsarajs.org/reference_docs/classes/Streams.Differential.html) is a stream that returns the deltas of the streams it subscribes from. This is useful for
getting deltas from a stream that doesn't normally supply them (like a `Transitionable`). 

```js
var differential = new Differential();
var transitionable = new Transitionable(0);

differentiable.subscribe(transitionable);

transitionable.set(100, {duration : 1000});

differentiable.on('update', function(delta){
    console.log(delta);
});
```

#### Combining Accumulators and Differentials

Let's have a `Surface` that can be dragged by the mouse, but springs back to
its original position when the user lets go of the mouse. Since the position of the `Surface`
is the combination of two sources - the mouse and spring transition - the easiest way to program
this is for the position to be an accumulator of the mouse and spring deltas. `MouseInput` emits
deltas by default, but a `Transitionable` must be fed into a `Differential` to get the deltas. The
following code demonstrates this.

<p data-height="266" data-theme-id="20796" data-slug-hash="MKXJzw" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/MKXJzw/'>transitionable-map-physics</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>