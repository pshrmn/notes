# D3 Axes

## About Axes

Axes are useful for displaying the scale of x or y axis. Scales can be quantitative, time based, or ordinal (qualitative such as a set of names). D3 provides methods for creating axes in the `d3-axis` module.

```bash
npm install d3-axis
```

## Create an Axis

D3 provides four methods, with their names indicating their alignment, for creating an axis generator: `axisTop`, `axisRight`, `axisBottom`, and `axisLeft`. A top aligned axis is horizontal and has ticks drawn above the axis. A bottom aligned axis is horizontal and has ticks drawn below the axis. A left aligned axis is vertical and has ticks aligned to the left of the axis. A right aligned axis is vertical and has ticks aligned to the right of the axis.

```js
import { axisBottom, axisLeft } from 'd3-axis';

const xAxis = axisBottom();
const yAxis = axisLeft();
```

An axis takes a scale function. The scale function is necessary to determine what values in the scale correspond to which pixels in the element that the chart is drawn in. The scale can either be passed to the axis when creating it or through its `scale` method.

```js
import { scaleTime } from 'd3-scale';

const xScale = scaleTime()
  .domain([new Date(1910, 0, 1), (new Date(2010, 0, 1))])
  .range([0, 500]);

const axis1 = axisBottom(xScale);
// or
const axis2 = axisBottom()
  .scale(xScale);
```

Remember that for vertical scales, 0 is the top of the `<svg>` so the range on a vertical scale should be reversed in most cases.

```js
import { scaleLinear } from 'd3-scale';

const yScale = scaleLinear()
  .domain([0, 10000])
  .range([250, 0]);
```

## Drawing Ticks

In order to know what values exist at points along an axis, ticks are used. These are rendered using a line and a text value.

The length of the lines drawn for each tick can be controlled. The ticks at the beginning and end of an axis are outer ticks, while all other ticks are inner ticks. Inner and outer ticks can be specified to have different lengths by using the `tickSizeInner` and `tickSizeOuter` methods. If you want both to have the same size, you can just use the `tickSize` method.

```js
yAxis.tickSizeInner(5);
yAxis.tickSizeOuter(10);

// or use tickSize to set both
xAxis.tickSize(5);
```

The axis has a `tickFormat` method that will be called when adding text to ticks. This allows you to modify the text value to something that looks better in the chart.

```js
import { format } from 'd3-format';
import { timeFormat } from 'd3-time-format';

xAxis.tickFormat(timeFormat('%Y'));
yAxis.tickFormat(format(',f'));
```

The distance between a tick and the text for the tick can be set using the `tickPadding` method.

```js
yAxis.tickPadding(5);
```

## Tick Values

If you don't specify how the axis should determine its ticks, this will be done for you. If you want greater control over these values, there are a number of methods that allow you to do so.

You can control the exact values of the ticks with the `tickValues` method.

```js
yAxis.tickValues([0, 75, 150, 1000, 2500, 5000, 10000]);
```

If you want to control how the ticks are determined without provided exact values, you can use the `ticks` method. This will use the axis's scale's `ticks` method to determine the tick values. It can take two arguments. The first argument is either the number of ticks that the axis should have (but not necessarily the number that it _will_ have, depending on how evenly they can be fit) or the time interval between ticks for a time scale. The second argument is a formatting function that will be used to render the tick's text value.

```js
yAxis.ticks(5);
```

The `tickArguments` function is similar to `ticks`, but takes the arguments as a single array.

```js
yAxis.tickArguments([5]);
```

## Rendering the Axis

To draw an axis, you need to create a group element for the axis. The axis doesn't know how to place itself, so you will need to transform the group to move it to the correct position. Then call the group element's `call` method, passing it the axis.

```js
improt { select } from 'd3-selection';

select('svg').append('g')
  .attr('transform', 'translate(50, 500)')
  .classed('x axis', true)
  .call(xAxis);

select('svg').append('g')
  .attr('transform', 'translate(100, 100)')
  .classed('y axis', true)
  .call(yAxis);
```

## Styling Axes
```css
/*
 * use crispEdges for a sharper looking axis
 * since these are vertical/horizontal lines
 * /
.axis line, .axis path{
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}

/*
 * align text with a tick
 */
.tick text{
  text-anchor: middle; /* or start or end */
}
```

## Full Example Code

Below is the full code used to render some x and y axes.

```js
import { axisBottom, axisLeft } from 'd3-axis';
import { select } from 'd3-selection';
import { scaleTime, scaleLinear } from 'd3-scale';

const holder = select(holder);
const fullWidth = 600;
const fullHeight = 250;
const margin = {
  top: 15,
  right: 15,
  left: 50,
  bottom: 50
};
const width = fullWidth - margin.left - margin.right;
const height = fullHeight - margin.top - margin.bottom;
const svg = holder.append('svg')
  .attr('width', fullWidth)
  .attr('height', fullHeight)
  .append('g')
    .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

const xScale = scaleTime()
.domain([new Date(1910, 0, 1), (new Date(2010, 0, 1))])
.range([0, width]);

const yScale = scaleLinear()
  .domain([0, 10000])
  .range([height, 0]);

const xAxis = axisBottom(xScale)
  .tickSize(5);
const yAxis = axisLeft(yScale)
  .tickSizeInner(5)
  .tickSizeOuter(10);

svg.append('g')
  .attr('transform', 'translate(0,' + height + ')')
  .classed('x axis', true)
  .call(xAxis);

svg.append('g')
  .classed('y axis', true)
  .call(yAxis);
```
