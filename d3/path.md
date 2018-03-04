# D3 Path

## About

The `d3-path` module is used to define how shapes are drawn inside of a `<path>` element. It has one main method, `path`, which returns an object that has methods for defining how to render the path.

```bash
npm install d3-path
```

## Path Holders

Before we look at what the `d3-path` module does, it is important to understand the HTML elements that can have paths rendered in them. Both the `<canvas>` and the `<svg>` contain similar ideas about how to define shapes using lines and curves.

### Canvas

In order to render a path to a `<canvas>` element, you need a context. The context contains a number of methods for drawing a path. When you begin to draw a path, you must start with moving to a certain pixel, which is done by calling the context's `moveTo` method. Then, you call a method to define how to move to another point and if a line or curve should be rendered between the two. You continue to call context methods until the path has been completed. Because we can programmatically define a path in JavaScript, we don't need to create a context when drawing a path inside of a canvas.

### SVG

Rendering a path inside of a `<svg>` element is done using its `d` attribute. This attribute is a string of commands concatenated together to define how the shape should be drawn. Each command starts with a letter and then a series of numbers. For example, `M 75,25` specifies that the path should move to the mixel with coordinates `<75,25>`. The SVG's rendering engine will parse this string to determine how to render the path. Manually creating the `d` attribute string would involve lots of string formatting and concatenation. `d3-path` exists to do this work for us, while using the same API as the canvas context.

## Path

The `path` method creates a new path object. This object acts as a context and has the same methods for drawing a path as the canvas's context. Because this object replicates the canvas context's method, we can write code that will render to either and svg or a canvas depending on the context.

```js
import { path } from 'd3-path';

const context = path();
context.moveTo(20,20);
context.lineTo(30,20);
// ...
```

The context object returned by the `path` method contains an array. Every time one of the object's rendering methods is called, the values representing how to render that step are added to the context object's array. Once every step needed to describe the path has been called, we can call the context object's `toString` method. This will concatenate the results together into a string that we then can use as the `d` attribute to a `<path>` element.

```js
// ...
const d = context.toString();
// d === 'M20,20L30,20'
```

## Context Agnostic Code

As mentioned above, the context object created by `d3-path` replicates the path rendering methods of a canvas context. That means that we can write code that takes a generic context object and it will render our path correctly regardless of whether it is to a canvas or an svg.

When using a canvas, we get a context object using the canvas's `getContext` method. When using an svg, we use `d3-path`'s `path` method.

```js
// given a canvas and svg element:
const canvas = document.querySelector('canvas');
const svg = document.querySelector('svg');
const path = svg.querySelector('path');

const canvasContext = canvas.getContext('2d');
const svgContext = path();

// we can then write element agnostic code
// to define how to draw the path
function drawSquare(context) {
  context.moveTo(10,10);
  context.lineTo(40,10);
  context.lineTo(40,40);
  context.lineTo(10,40);
  context.closePath();
}
```

This doesn't contain any magic that gets rid of other rendering related code that differs between the two types. For canvas elements, we will still need to call the context's `beginPath` method to begin the drawing and either its `stroke` or `fill` method to finish it. For svg elements, the context will need to be output as a string and set as the `d` attribute. Still, all of the steps in between can be shared.

```js
// render to the canvas
canvasContext.beginPath();
drawSquare(canvasContext);
canvasContext.stroke();

// render to the svg
drawSquare(svgContext);
path.setAttribute('d', svgContext.toString());
```
