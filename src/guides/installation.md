## Starter Kit

The simplest way to download Samsara is by clicking this button!

<a id="download" href="https://github.com/dmvaldman/samsara/archive/v0.2.0.zip">
	<div id="title">Starter Kit</div>
	<div id="version">v0.2.0</div>
</a>

Inside you'll find:

| Directory | Description |
| --------- | ----------- |
| samsara | AMD source files. Samsara is currently developed using require.js. |
| dist | CommonJS bundle. Useful for building with browserify, or including directly into an HTML document. |
| examples | Documented examples. These use the AMD files and require.js |
| docs | API Reference documentation | 

Remember to also include the `samsara.css` file in any website using Samsara.
This file is located in `dist/samsara.css` and in `samsara/samsara.css`.

## NPM

Install the CommonJS build of Samsara with

```
npm install samsarajs
```

## GitHub

Clone Samsara from GitHub
 
```
git clone git@github.com:dmvaldman/samsara.git
```

This is the preferred method for development on Samsara and remains the most up to date.

## Window Object

To expose `Samsara` on the browser's `window` object, include the samsara.js bundle in a `<script>` tag. The files can
be found in the starter kit's `dist` folder, or on [GitHub](https://github.com/dmvaldman/samsara/tree/master/dist). 

```
<script type="text/javascript" src="path/to/samsara.js"></script>
<link rel="stylesheet" type="text/css" href="path/to/samsara.css"/>
```

`Samsara` acts as a namespace for all of Samsara's components. For instance, you'll find `Surface` in 
`window.Samsara.DOM.Surface`. 

### Namespace Differences

There is a capitalization difference between the AMD and CommonJS and `window` namespaces. For
AMD, all but the filename is lowercase, and for CommonJS and the `window` object, everything is uppercase.
 
```js
var Transitionable = require('samsara/core/Transitionable')   // AMD
var Transitionable = require('samsarajs').Core.Transitionable // CommonJS
var Transitionable = Samsara.Core.Transitionable              // window object
```
 
Currently we bundle everything up into the `samsarajs` namespace for the CommonJS format, though this may
change in the future if users would like a modular CommonJS build of Samsara.
 