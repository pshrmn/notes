# D3

D3, which stands for data driven documents, is made up a number of JavaScript modules that aid in making data visualizations.

**Note:** The word data is the plural of datum. When data is used, it is referring to an array of datum.

D3's modules have three main functionalities:

## Preparation

D3 can analyze your data and add properties to it in order to prepare it to be rendered. 

For example, layouts are used to add properties to each datum. A pie layout will take a data array and return a new array where each datum will contain start and end angle information that can be used to draw the arcs for a pie chart.

```js
const data = [{value: 1}, {value: 3}, {value: 2}, {value: 7}];
const pieData = pie(data);
```

## Transformation

D3 provides a number of modules that add support for rendering.

Scale functions help you transform properties of your data into values used as attributes or text for DOM elements.

```js
const scale = d3.scaleLinear()
  .domain([0, 10])
  .range([0, 100])
//scale(1) === 10
```

Shape generators assist in creating the paths used to draw elements.

```js
const line = d3.line();
  .x(d => xScale(d.x))
  .y(d => yScale(d.y))
  .curve(d3.curveCardinal);
```

## DOM Manipulation

Using something called selections, D3 can work with the DOM to insert elements into a page, update them, and remove them. Selections are made by binding your data to an array of DOM elements. Selections can use transitions to animate DOM changes the bound data is updated.

```js
const data = ['one', 'two', 'three'];
d3.select('#root').selectAll('div.thing')
    .data(data)
  .enter().append('div')
    .classed('thing', true)
    .text(d => d)
```

The preparation and transformation methods do not care if you are using D3's DOM manipulation methods. You can interact with the DOM any way that you like, whether that is with D3 selections, React or any other of a number of libraries, or manually creating elements. This decoupling of functionality is what makes D3's splitting into multiple modules in version 4 great. You only need to include the parts of D3 that are necessary to your project, which reduces the amount of data that users of your project need to download.


While you can use any DOM elements with D3, `<svg>` elements and the `<canvas>` provide the most flexibility in creating visualizations. Using canvases and svg elements, you are unconstrained by the CSS box model.

If you are unfamiliar with either the `<svg>` or `<canvas>`, check out the [svg](svg.md) and [canvas](canvas.md) overviews.
