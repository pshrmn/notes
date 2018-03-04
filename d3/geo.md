# D3 Geo

## About

The `d3-geo` module is used to draw GeoJSON and TopoJSON features. It contains a path generator to create the path that outlines a geographic element and projection functions that control how coordinates in the GeoJSON feature map to pixels in an `<svg>` or `<canvas>` element. The module also has sphere related methods, which can be helpful when projecting to a globe, and graticule methods, which draw lines representing latitudes and longitudes.

```bash
npm install d3-geo
```

## GeoJSON & TopoJSON

A common way to store geographic information is using the GeoJSON format. GeoJSON files describe 'features', which are made up of polygons, points, and lines. For example, a GeoJSON object that describes how to render all of the states in the United States would contain a feature for every state. The feature for a state would be a polygon made up of a series of coordinates which define the border of the state.

One problem with GeoJSON is that it can contain a lot of redundant data. For example, most states in the United States share borders. In GeoJSON, each of these shared borders would be replicated for each state that contains it. TopoJSON removes this redundancy by creating arcs (arrays of coordinates). A polygon is made up of arcs, so states with a common border will both contain the same arc.

## Path Generator

The `geoPath` method returns a path generator. This generator will be passed feature data (from a GeoJSON or TopoJSON object) to output directions on how to render a path.

```js
import { geoPath } from 'd3-path';
const path = geoPath();
svg.select('svg').append('path')
  .datum(featureData)
  .attr('d', path);
```

### Projections

There are a number of projection functions to render geographic elements differently. The default projection used by the `geoPath` generator is null, which simply maps geographic features using the provided values.

Among the included projections are <a href="https://github.com/d3/d3-geo#geoMercator">Mercator</a>, <a href="https://github.com/d3/d3-geo#geoAlbersUsa">Albers</a>, and <a href="https://github.com/d3/d3-geo#geoOrthographic">Orthographic</a> projections. The projections can be configured to change how they render the path. The <a href="https://github.com/d3/d3-geo#geoAlbersUsa">Albers USA</a> projection is a pre-configured Albers projection that draws a map of the continental 48 states with Alaska and Hawaii placed in the lower left corner.

```js
import { geoAlbersUsa } from 'd3-geo';
const path = geoPath()
  .projection(geoAlbersUsa());
```

A few of the projection configuration methods are listed below. For a complete list, check out the <a href="https://github.com/d3/d3-geo#geoProjectionMutator">projection documentation</a>.

The `clipExtent` projection method takes an array of two coordinates. The coordinates make up a rectangle, with the first being the top left corner and the second the bottom right corner. Any parts of the path that are outside of this rectangle will be cut off.

```js
import { geoAlbers } from 'd3-geo';
const projection = geoAlbers()
  .clipExtent([0,0], [200,200]);
```

The `scale` method allows you to control the scaling for converting coordinates to pixels. Each projection has its own default scale.

```js
const projection = geoAlbers()
  .scale(500);
```

The `translate` method takes an array with two values. The projection's output path will be horizontally translated by the first value and vertically translated by the second.

```js
const projection = geoAlbers()
  .translate([10, 10]);
```

The projection's `fitExtent` method takes a bounding rectangle and a feature object and determines the projection's scale and translation for you. The `fitSize` method is nearly the same, but instead of tale the corners of a bounding rectangle, it expects the width and height of the rectangle.

```js
// given a GeoJSON feature called "feature"
const projection = geoAlbers()
  .fitExtent([[10,10], [190,190]], feature);
```

### Context

The context of the path generator is used to generate the path. If you do not provide a context, this will output an svg `<path>` string (the value for the `d` attribute). Otherwise, you can pass it a canvas context to draw the feature to a `<canvas>`.

```js
const canvas = document.querySelector('canvas');
const context = canvas.getContext('2d');

const path = geoPath()
  .context(context);
```

### Point Radius

When the path has to draw a point, it draws a circle. You can control the radius of this circle with the `pointRadius` method. This can either be a number or a function that will determine a radius based on the feature. The default radius is `4.5`.

```js
const path = geoPath()
  .pointRadius(6);
```

## Graticules

Graticules help visualize relative latitudes and longitudes of points within a projection.

```js
import { geoGraticule } from 'd3-geo';
const graticules = geoGraticule();
const path = geoPath();
svg.select('svg').append('path')
  .datum(graticules)
  .attr('d', path);
```

The `extent` of the graticules can be controlled. This controls the coordinates of northern-, southern-, western, and easternmost points within which graticules should be drawn. These values are provided in degrees.

```js
const graticules = geoGraticule()
  .extent([[-90, -90], [90, 90]]);
```

The number of degrees separating the graticule lines is defined using line methods. These values are provided in degrees.

```js
const graticules = geoGraticule()
  .step([45, 30]);
```
