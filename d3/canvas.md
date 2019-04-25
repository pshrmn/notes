# `<canvas>` Overview

Just like a physical canvas, the HTML `<canvas>` element is made to be drawn on. The drawing on a `<canvas>` is done using a context object, unlike an `<svg>`, which uses nested elements. Still, the `<canvas>` does not use a self-closing tag. This allows for any text or elements placed inside of the canvas element to be rendered when the browser cannot render the canvas.

```html
<canvas>This only displays if the browser cannot render a canvas</canvas>
```

In order to draw on a canvas, you need a context object. You get a context by calling the `getContext` method on the `<canvas>` element. The context that you will probably want is `'2d'`, but a `'webgl'` context also exists for three dimensional drawing. A context object has a number of methods that determine where and how to draw pixels on the canvas. It has also style values that can be set to change what the drawn pixels look like (for both its fill and stroke) and transformation values that affect the placement of the shapes.

```js
const canvas = document.querySelector('canvas');
const context = canvas.getContext('2d');

context.fillStyle = 'red';
```

**Note:** For a full list of the 2d context's methods, check out its <a href="https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D">documentation</a> on MDN. The <a href="https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D#Paths">path</a> methods are used to draw paths similarly to how the SVG path is drawn, but with methods instead of a string.

Whenever the context draws to the canvas, it uses the style values it currently has set on it. Similarly, transformations can be made on the context to alter where items are drawn. The context has `save` and `restore` methods, which allow you to safely alter and restore the style and transformation values.

```js
context.strokeStyle = 'blue';
context.fillStyle = 'orange';
```

Whereas the `<path>` element of the `<svg>` was defined using a configuration string, for a `<canvas>`, a path is defined using a series of context method calls.

```js
// a path is defined using a configuration string
const path = document.createElement('path');
path.setAttribute('d', 'M50,50 L25,75');

// a path is defined using method calls
const canvas = document.createElement('canvas');
const context = canvas.getContext('2d');
context.beginPath();
context.moveTo(50, 50);
context.lineTo(25, 75);
// ...
```

After you have finished defining a path, you will need to call the context's `stroke` and/or `fill` methods to draw the stroke and/or fill for the path.

```js
// ...
context.stroke();
```

Unlike an `<svg>`, you are not creating elements that can be modified and removed. Any time that you want to remove or update something that you have drawn to the canvas, the whole thing will need to be cleared and redrawn. While this might seem slow, the canvas is actually very fast. In fact, people use the canvas to render games that need to redraw many times per second.

```js
context.clearRect(0, 0, width, height);
```
