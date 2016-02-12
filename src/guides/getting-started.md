## Draw a Square

Let's put a square on the screen. Here we'll be introduced to two Samsara primitives: 
[`Surface`](http://samsarajs.org/reference_docs/classes/DOM.Surface.html) and 
[`Context`](http://samsarajs.org/reference_docs/classes/DOM.Context.html). 

* _A **`Surface`** wraps a DOM element in which you can place arbitrary HTML content. By default `Surfaces`
appear as `<div>` tags (though any valid HTML tag can be substituted)._

* _A **`Context`** establishes a 3D environment in CSS so that perspective and 3D transforms behave 
appropriately. It has no visual representation, but is the root from which all other content is rendered._

`Contexts` are mounted to existing DOM nodes, and all `Surfaces` added to them show up in HTML as a nested elements.
To get a square on the screen, we represent the square as a `Surface` and provide it a size, content, basic CSS
properties, and add it to a `Context`.

```js
var Surface = Samsara.DOM.Surface;
var Context = Samsara.DOM.Context;

var surface = new Surface({
    content : 'hello',      // innerHTML
    size : [100, 100],      // [width, height] in pixels
    properties : {          // CSS style properties
        background : 'red'
    }
});

var context = new Context();

context.add(surface);
context.mount(document.querySelector('#myApp'));
```

<p data-height="268" data-theme-id="20796" data-slug-hash="xwyLpe" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/xwyLpe/'>getting-started-1</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

Starting from a `Context` and adding nodes builds up the render tree: a JavaScript representation of the layout of a page.
Read more about the render tree [here](render-tree.md).

## Simple Layout

Let's place this square somewhere else. To do so, we'll see we can add more than just `Surfaces` to a `Context`. 
`Surfaces` are the leaves of the render tree, everything between them and a `Context` are nodes that modify the 
layouts and sizes of the `surfaces` beneath them.

* _A **node** is a JSON object with layout data. The valid keys are: `transform`, `align`, `origin`, `opacity`, `size`, 
`proportions`, `margins`, and `aspectRatio`_. 

You can read more about nodes and the values they accept [here](render-tree.md#nodes). Here's an example where
we add a node with a `transform` property that takes a Samsara [`Transform`](http://samsarajs.org/reference_docs/classes/Core.Transform.html)
to translate the `surface` to the right by 100px.

* _A **`Transform`** corresponds to [CSS3 transforms](https://developer.mozilla.org/en-US/docs/Web/CSS/transform), 
allowing you to translate, rotate, skew and scale DOM elements. `Transforms` ultimately end in in-lined CSS properties
for the `Surfaces` they affect._

```js
var Transform = Samsara.Core.Transform;

context
    .add({transform : Transform.translateX(100)})
    .add(surface);
```

<p data-height="266" data-theme-id="20796" data-slug-hash="dYgzQo" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/dYgzQo/'>getting-started-3</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script> 
<br>

You can experiment with other `transforms` like `Transform.skewX(Math.PI/4)`, `Transform.scaleY(2)`, etc.
`Transforms` can also be compounded together with the `Transform.compose` and `Transform.composeMany` methods.

```js
var scaleAndRotate = Transform.compose(
    Transform.scaleX(2),
    Transform.rotateY(Math.PI/4)
);
```

## Simple Animation

We've been creating `Surfaces` with a static size set to `[100, 100]`. Many properties in Samsara can take dynamic values. 
To change a value over time we can use a [`Transitionable`](http://samsarajs.org/reference_docs/classes/Core.Transitionable.html).

* _A **`Transitionable`** represents a number or array of numbers that changes over time. Values can be interpolated using
easing curves, and physical transitions like springs._

You can read more about `Transitionables` [here](animation.md). In this example, we animate the size from `[100, 100]` to 
`[200, 200]` with an easing curve.

```js
var size = new Transitionable([100, 100]);

var surface = new Surface({
    content : 'click me',
    size : size,
    properties : {background : 'red'}
});

surface.on('click', function(){
    size.set([200, 200], {curve : 'easeOutBounce', duration : 1000});
});
```

<p data-height="266" data-theme-id="20796" data-slug-hash="BoqdrW" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/BoqdrW/'>BoqdrW</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>

Other properties on `Surface` that can take streaming values are: `origin`, `opacity`, `proportions`, `margins` and `aspectRatio`.

## Compound Animation

Let's rotate the `Surface` as we animate its size. We'll use a physics transition for the rotation.
A rotation is a type of `Transform`, and `Transitionables` don't directly output `Transform` types. 
However, `Transitionables` have a `map` method, which we will use to convert values to `Transforms`. 

```js
var size = new Transitionable([50, 50]);
var angle = new Transitionable(0);

var surface = new Surface({
    size : size,
    properties : {background : 'red'},
    origin : [.5,.5]   // sets the "origin" point to the center of the surface
});

surface.on('click', function(){
    size.set([150, 150], {curve : 'easeOutBounce', duration : 1000});
    angle.set(Math.PI, {curve : 'spring', period : 100, damping : .3});
});

// map the transitionable's value to a Transform
var rotation = angle.map(function(angle){
    return Transform.rotateZ(angle);
});

var context = new Context();

context
    .add({
        transform : rotation,
        align : [.5,.5] // aligns the origin point of the surface with the center of the context
    })
    .add(surface);

context.mount(document.querySelector('#myApp'));
```

<p data-height="266" data-theme-id="20796" data-slug-hash="QyqLLW" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/QyqLLW/'>getting-started-compound-animation</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>
<br>

Above we sneakily introduced the concepts of `align` and `origin` in order to center the square. Read more about
these layout primitives [here](render-tree.md#nodes).

## Synchronized Animation

One of SamsaraJS's core principles is to make synchronizing animation simple. 
In the following example we will grow the size of one `Surface` while keeping another `Surface` adjacent to it. 

```js
var t = new Transitionable(1);

// map the transitionable to a size for the red surface
var redSize = t.map(function(value){
    var length = 100 * value;
    return [length, length];
});

// map the transitionable to a `Transform` for the blue surface
// this will offset it in sync with the growing size of the redSurface
var blueTransform = t.map(function(value){
    return Transform.translateX(100 * value);
});

var redSurface = new Surface({
    content : 'click me',
    size : redSize,
    properties : {background : 'red'}
});

var blueSurface = new Surface({
    size : [100, 100],
    properties : {background : 'blue'}
});

redSurface.on('click', function(){
    t.set(2, {duration : 1000, curve : 'easeOutBounce'});
});

var context = new Context();

// build the render tree
context.add(redSurface);
context
    .add({transform : blueTransform})
    .add(blueSurface);

context.mount(document.getElementById('myApp'));
```

<p data-height="266" data-theme-id="20796" data-slug-hash="QjZMJQ" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/QjZMJQ/'>getting-started-4</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>
<br>

There's another way to to this in SamsaraJS as well. `Surface's` have a `size`
property which is a stream. So you could replace `blueTransform` with

```js
var blueTransform = redSurface.size.map(function(size){
    return Transform.translateX(size[0]);
});
```

This will work without even if the size of the `redSurface` is animating, and avoids the need to hardcore the value 100
in multiple places.

### Motion → Layout

At the expense of pedagogy, we're going to jump ahead a few steps. We'll use user input from the mouse instead of a 
`Transitionable` in the above example.  We'll allow the `redSurface` to be draggable, and have its size increase with 
the drag behavior, all-the-while keeping the `blueSurface` adjacent.

<p data-height="350" data-theme-id="20796" data-slug-hash="EVdvOJ" data-default-tab="result" data-user="samsaraJS" class='codepen'>See the Pen <a href='http://codepen.io/samsaraJS/pen/EVdvOJ/'>EVdvOJ</a> by SamsaraJS (<a href='http://codepen.io/samsaraJS'>@samsaraJS</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="http://assets.codepen.io/assets/embed/ei.js"></script>
<br>

## Wrap Up

This scrapes the `surface` of what SamsaraJS can do. Read through more of the guide, explore the API in depth
from the [reference documentation](http://samsarajs.org/reference_docs/index.html), or take a look at the 
[examples](https://github.com/dmvaldman/samsara/tree/master/examples) to see more practical applications.