# `<svg>` Overview

While D3 can use any type of HTML elements, one of the most common use cases is SVG elements. SVG elements (alongside the `canvas`) are used to make a majority of the D3 chart visualizations.

**Note:** There are a number of examples in this tutorial that should sample code and an SVG with the rendered contents. However, the sample code is not always the full code used to render, but just includes information being described. You can inspect the SVG using developer tools (right click on the SVG and select Inspect/Inspect Element) to see the full code used to render the SVG.

## What is SVG?

The SVG language is an XML-like markup language, meaning that it defines elements using tags, which can have attributes set for them as well as child elements. These elements are then rendered to create an image.

The G in SVG is Graphics. SVGs are used to create things that are meant to be rendered as images. In D3, this is often used to make charts. Graphics that need to be scaled to many different sizes, such as logos, are often created using SVGs.

The V in SVG is Vector. In SVG, everything is defined mathematically.

For example, you might want to create a graphic with a line segment. There are multiple ways to draw this with SVGs, but the simplest would be to use a `<line>` element. The line segment is defined using two points, for example `<0, 0>` and `<50, 50>`. When the SVG is rendered,  a line segment is drawn extending from the first to the second point.

```html
<svg width="100" height="100">
  <line x1="0" y1="0" x2="50" y2="50"></line>
</svg>    
```

**Note:** The origin `<0, 0>` for an SVG is in the top left corner. The point at the bottom right corner is `<width, height>`, based on the width and height of the SVG.

The S in SVG is Scalable, which references the fact that you can transform SVGs (such as scaling them) without a loss in quality. Because each element in the SVG is defined using numbers, we can use math to modify the elements using transformations. In fact, these can be chained together to create more complex transformations.

If we want to double the size of the line in our image, we just scale our line segment by multiplying our points by two. We then end up with the new points `<0*2, 0*2> = <0, 0>` and `<50*2, 50*2> = <100, 100>`

```html
<svg width="100" height="100">
  <line x1="0" y1="0" x2="50" y2="50" transform="scale(2)"></line>
</svg>    
```

### Comparison with Raster Graphics

Raster graphics are graphics where a color value is assigned to every pixel in the graphic. When you take a picture, that is creating a raster graphic. Raster graphics are great for many things, but they don't lend themselves to being transformed like vector graphics do.

Imagine our line graphic above. In the raster version of the graphic, most of the pixels would be mapped to the white value, but the ones where the line is located would be assigned the black value. What happens when we want to scale the size of the line in the image? A new image would be created and every pixel in the new image would need to be assigned a value based on a corresponding value in the original image, then we would crop the image back down to its original size, which would then contain the properly scaled line.

This is relatively simple, but what if the image contains a line and a circle and we only want to transform the circle. This is simple using SVGs but would require lots of manual editing in raster graphics.

## Defining Elements

The way that elements should be rendered is defined via attributes. The majority of attributes can be split into two categories: those that define how to render the element geometrically and those that define how to render the element graphically. Given a `<circle>`, the geometric attributes would be the location of its center and its radius, while its graphic attributes could describe what its `fill` color is and how thick its `stroke-width` is. <a href="https://developer.mozilla.org/en-US/docs/Web/SVG/Element">Mozilla</a> provides a good reference to which attributes are used to style different elements.

Below are examples of a few commonly used attributes.

### Geometric Attributes:

##### `width`

The width of an element.
```html
<rect width="25" />
```

##### `height`

The height of an element.

```html
<rect height="25" />
```

##### `transform`

The transformations to apply to an element. When chained together, these will be executed from right to left.

```html
<rect transform="scale(2)" />
```
   
##### `x`, `cx`, `x1`, `x2`

A location on the x-axis.

```html
<rect x="10" />
<line x1="10" x2="20" />
<circle cx="15" />
```

##### `y`, `cy`, `y1`, `y2`

A location on the y-axis.

```html
<rect y="10" />
<line y1="10" y2="20" />
<circle cy="15" />
```

##### `d`

A description of how to draw a path. Basically, the string is made up of commands concatenated together. Each command starts with a letter, which specifies the type of the command and then a series of numbers that are used by the command. The numbers are usually either coordinates or lengths, but sometimes also describe other attributes for the command. The command letter can either be upper or lowercase. When uppercase, numbers are coordinates, and when lowercase, numbers are lengths from the current position. For a full explanation of how this works, read the <a href="https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/d">documentation</a> on Mozilla Developer Network.

```html
<path d="M10,10 l10,5" />
```
   
### Graphic Attributes:

When drawing an element, the interior of the element is styled by the `fill` and the border of the element is styled by the `stroke`.

##### `fill`

The color that an element should be filled with. This can also be a gradient or a pattern.

```html
<rect x="25" y="25" width="50" height="50" fill="blue" />
```

##### `stroke`

The color that should be used on the border of the element. This can also be a gradient or a pattern.

```html
<rect x="25" y="25" width="50" height="50" stroke="blue" />
```

##### `stroke-width`

How thick a stroke should be. When an element is transformed, the thickness will also be tranformed. SVG 1.2 adds a `vector-effect="non-scaling-stroke"` value which prevents this.

```html
<rect x="25" y="25" width="50" height="50" stroke="blue" stroke-width="3" />
```

##### `text-anchor`

The text anchor determines how `<text>` is placed in respect to the element's origin (the coordinates where the text is placed in the SVG). To picture how this is work, imagine a rectangle used to bound the text. When text-anchor is `start`, the left edge of the rectangle is placed at the origin. For `middle`, the rectangle is centered on the origin, with an equal length to the left and right of the point. For `end`, the right edge of the rectangle is placed at the origin.

```html
<text text-anchor="start">start</text>
<text text-anchor="middle">middle</text>
<text text-anchor="end">end</text>
```

##### `shape-rendering`

When an SVG is being rendered, the rendering engine makes decisions about how the pixels for an elemente are mapped to pixels in the image. Sometimes you will want to provide hints to the engine about how certain elements are rendered. The `shape-rendering` attribute lets you do that, either for the entire graphic by adding it to the `<svg>` element or for individual elements.

When the rendering engine is rendering elements, a pixel from an element might not directly align with a single pixel in the image, but instead is split between two pixels. This is anti-aliasing and works well for curves and diagonal lines. However, for vertical and horizontal edges, this can result in a blurry appearance. The `shape-rendering="crispEdges"` value helps combat this because it tells the rendering engine to forces elements to be aligned with a pixel. This should not be used for curved shapes unless you want really ugly shapes.

```html
<line x1="0" x2="100" y1="25" y2="25" />
<line x1="0" x2="100" y1="75" y2="75" shape-rendering="crispEdges"/>
```
The other possible values for `shape-rendering` are: `auto` (default) which lets the rendering engine do what it thinks is best, `optimizeSpeed` which tells the rendering engine to render as fast as possible, and `geometricPrecision` which tells the rendering engine to make the image as accurate as possible.

## Common SVG Elements

#### `<svg>`

When creating an SVG, the root element is `<svg>`.

```html
<svg></svg>
```

The size of the SVG is controlled using the `width` and `height` attributes. This has to be done through attributes, CSS will not affect the SVG.

```html
<svg width="100" height="50" ></svg>
```
#### `<g>`

A `<g>` element is used to create a group. The `<g>` element isn't rendered visually, it is just there for grouping/placing its child elements. A `<g>` element can be transformed so that all of its children are rendered based on its location. You can also set style attributes on the `<g>` that will be inherited by its child elements.

```html
<g>
  <childElement />
  <childElement />
</g>
```

#### `<line>`

A `<line>` is a line segment made by connecting two points.

```html
<line x1="10" y1="10" x2="80" y2="57" />
```

#### `<rect>`

A `<rect>` draws a rectangle given the coordinates for its top left corner, its width, and its length.

```html
<rect x="12" y="17" width="70" height="47" />
```

#### `<circle>`

A `<circle>` is drawn using the coordinates of its center and its radius.

```html
<circle cx="50" cy="40" r="27" />
```

#### `<text>`

Text is added to an SVG using the `<text>` element.

```html
<text>This is the text</text>
```

#### `<path>`

A `<path>` element is used to draw any shape that you want. The shape is defined using the `d` attribute.

```html
<path d="M40,20 l20,0 l0,20 l20,0 l0,20 l-20,0 l0,20 l-20,0 l0,-20 l-20,0 l0,-20, l20,0 l0,-20" />
```
