# D3 Layouts

D3 has a number of layout functions. Layout functions sets up and modifies data so that it can be rendered in different shapes. The layout functions don't do the drawing of the shapes.

## Pie Layout

The pie layout is a part of the `d3-shape` module. 

```bash
npm install d3-shape
```

A pie layout transforms data by adding start and end angle values to each item in the data array so that it can be rendered as a pie chart. It is used in conjunction with a `d3.arc` (also part of `d3-shapes`) to draw the pieces of the pie. If the arc function is passed an inner radius, the pie chart can be rendered as a donut chart.

The pie layout has a number of methods that you can use to control how the start and end angles are determined.

```js
import { pie } from 'd3-shape';

const pie = d3.pie()
  // set how to get the value being charted from each datum
  .value(function(d){ return d.count; })
  // set the sort function (or null to leave in original order)
  .sort(null)
  // set the start angle of the first datum in radians (default 0)
  .startAngle(Math.PI/2)
  // set the end angle in radians (default 2*Math.PI)
  .endAngle((5/2)*Math.PI)
  // set padding between arcs in radians (default 0)
  .padAngle(0);
```

If you want your pie chart to be a full circle, the difference between the start angle and the end angle should be 2π radians.

Once your pie layout function has been configured, pass it your data array. This will return an array of objects which have the `data` for the corresponding datum as well as `startAngle` and `endAngle` properties.

```js
// mock data
const data = [
  {name: 'US', count: 11},
  {name: 'Germany', count: 5},
  {name: 'United Kingdom', count: 7},
  {name: 'Canada', count: 13},
  {name: 'Norway', count: 4}
];

// transform our data to have relevant properties
// for rendering a pie chart
const pieData = pie(data);
```

The pie chart is made up of arcs which are drawn using an `arc` generator. The arc generator can be configured using a number of methods, but the two most important are `outerRadius` and `innerRadius`. The `outerRadius` method defines the radius of the pie chart. The default inner radius is 0, but you can provide a number larger than that using the `innerRadius` method to create a donut chart.

```js
import { arc } from 'd3-shape';

// create an arc function to calculate angles
// for arcs in the data
const arc = arc()
  .outerRadius(100)
  .innerRadius(0);
```

Once our arc generator has been configured, create a selection of `&lt;path&gt;` elements and bind it to the data array returned by the pie layout function. The arc generator function is used to create the `d` attribute of each path.

```js
svg.selectAll('path.arc')
    .data(pieData)
  .enter().append('path')
    .classed('arc', true)
    .attr('d', arc);
```

## Tree Layout

The tree layout is part of the `d3-hierarchy` module.

```bash
npm install d3-hierarchy
```

Tree layouts can be useful for displaying nested information. In order to create a tree, you will need to have nested data and also use both the `hierarchy` and `tree`  functions.

### `hierarchy` function

The `hierarchy` function takes your nested data, creates a `Node` object for each datum, and returns the root `Node`. It can also take a second argument, which is a function that returns a datum's children. It is only necessary to include this if the child datum aren't in the datum's `children` property.

Each `Node` created by the `hierarchy` function has properties that will be used by the `tree` function to create the layout:

* `data` - The datum object from your data that is associated with this node.
* `depth` - The depth of the node. The root node has depth 0, its children have depth 1, and so on and so forth.
* `height` - The height of the node is the longest distance from the node to a leaf node (a node with no children). A leaf node has a height of 0.
* `parent` - A reference to the node's parent node.
* `children` - An array of child nodes for a node. This only exists if the node has child nodes.

```js
import { hierarchy } from 'd3-hierarchy';

const data = {
  name: 'parent',
  children: [
    {name: 'child1'},
    {name: 'child2'}
  ]
};

const root = hierarchy(data);
/*
 * root === Node(data={name: 'parent'}, ...)
 */
```

The `Node` object has a number of methods. The most important of these for a tree layout are the `descendants` and `links` methods. The `descendants` method returns an array containing all the node and every descendant node. Calling this on the root node will give you an array of every node. The `links` method creates an object for each connection between a parent and a child node in the tree. In each object, the parent node is the `source` and the child node is the `target`.


```js
const descendants = root.descendants();
/*
 * descendants === [Node(...), Node(...), Node(...)]
 */
const links = root.links();
/*
 * links === [
 *  {
 *    source: Node(data={name: 'parent'}, ...)
 *    target: Node(data={name: 'child1'}, ...)
 *  }, ...
 * ]
 */
```

### `tree` function

The `tree` function takes a `Node` object and calculates the position for it and all of its descendant nodes in the tree. It does this by creating a `TreeNode` for each node and then walking through the tree nodes to determine the positioning of each, adding those values to the nodes.

The tree layout can be controlled using its `separation`, `size`, and `nodeSize` methods. The `separation` method takes a function which returns the amount of separation there should be between two nodes drawn near each other. `size` takes an array containing the width and height of the tree. For a vertical tree (top-down), the array should be `[width, height]`. For a horizontal tree (left-right), the array should be `[height, width]`. `nodeSize` is used to specify the size of each node using an array containing the width and height of each node. For the `size` and `nodeSize` methods, only one can be active on a tree. If both are called, the second one will be the one that is used.

```js
import { tree } from 'd3-hierarchy';

const tree = tree()
  .size([200, 400]);
tree(root);
```

### Drawing the Tree

Now that you have your data setup to render a tree, its time to actually draw it. You will need to draw two things: the nodes and the links between the nodes.

To draw the nodes, call the `descendants` method of the root node to get an array of all of the nodes.

```js
const nodes = root.descendants();
```

Each node has an `x` and `y` property. You can use these to translate each node to the correct position. The `x` and `y` properties don't know the orientation of your tree. If you are using a vertical tree, you can use them as expected, but for a horizontal tree the values will have to be swapped.

```js
svg.selectAll('g.node')
    .data(root.descendants())
  .enter().append('g')
    .classed('node', true)
    // for a horizontal tree, d => `translate(${d.y},${d.x})
    .attr('transform', d => `translate(${d.x},${d.y}))
```

To draw the links, call the `links` method of the root node. This will return an array of objects, each one having a `source` and a `target` node. You can then generate a path between the two nodes.

```js
function drawLink(node) {
  const context = d3.path();
  const { source, target } = node;
  context.moveTo(target.y, target.x);
  context.bezierCurveTo(
    (target.y + source.y)/2, target.x,
    (target.y + source.y)/2, source.x,
    source.y, source.x
  );
  return context.toString();
}

svg.selectAll('path')
    .data(root.links())
  .enter().append('path')
    .style('fill', 'none')
    .style('stroke', '#444')
    .attr('d', drawLink);
```
