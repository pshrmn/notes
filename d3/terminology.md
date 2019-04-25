# D3 Terminology

* `datum` - an individual unit of information
```js
const datum = {x: 10, y: 5};
```
* `data` - a collection of datum
```js
const data = [{x: 10, y: 5}, {x: 7, y: 7}, {x: 1, y: 9}];
```
* `selection` - an array of DOM elements. A selection has an array of data bound to it. Each item in the array is bound to an element. When there are fewer elements that items, new elements are made. When there are more elements than items, unmatched elements are removed from the DOM.
```js
const selection = d3.select('svg').selectAll('circle');
```
* `extent` - A two element array containing the lower and upper bounds for a set of values.
```js
const values = [1,7,3,6,5,8,2];
const extent = d3.extent(values);
// extent === [1, 8]
```
* `scale` - A function used to map data from an input domain to an output range. In most cases, the default domain and range are both [0, 1].
* `domain` - Upper and lower limits for input data for a scale. The lower limit will be mapped to the lower limit of the range and the upper limit will be mapped to the upper limit of the range. Values outside of the domain can be provided to a scale, but these typically will appear outside of the svg if the range is the width of the svg.
```js
const line = d3.scale.linear()
  .domain(0, 100);
const min = line(0); // === 0
const max = line(100); // === 1
const outside = line(200); // === 2
```
* `range` - Upper and lower limits for input domain to be mapped to in a scale. The lower limit of the domain maps to the lower limit of the range and the upper domain limit maps to the upper range limit.
```js
const line = d3.scale.linear()
  .range([10, 20]);
const min = line(0); // 10
const max = line(1); // 20
const outside = line(2); // 30
```
* `interpolate` - To map data from [0,1] (representing a percent, from 0% to 100%) to an output range, based on the type of the interpolator. Different types include numbers, colors, and objects.
```js
// interpolate from black to white
const grayscale = d3.interpolateRgb('#000', '#fff');
// get the color halfway between black and white
const gray = grayscale(0.5);
// gray === '#808080'
```
* `continuous` - Given a lower and an upper boundary, a continuous set includes every possible number between (and including) the boundaries. For example, the continuous set with a lower boundary of 0 and an upper boundary of 10, the set includes 0, 0.00001, 1, 3.14, 7, 10, and every other number that is equal to or larger than 0 and also less than or equal to 10. A continuous set is always quantitative.
* `discrete` - A discrete set of data is an explicit set of values. This can be quantitative (the set of whole numbers with boundaries 0 and 5 is {0,1,2,3,4,5}) or categorical (the set of months is {January, February, March, April, May, June, July, August, September, October, November, December}).
