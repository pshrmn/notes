# D3 Selections

D3 uses selections to interact with the DOM. While D3 version 4 transformed it into many modules, allowing you to use only select pieces of it, selections were the lifeblood of the original library. While they aren't necessary to use D3, selections are very efficient at manipulating the DOM and are important to understand.

```bash
npm install d3-selection
```

## Making Selections

A selection is made up of one or more DOM elements in the page. It is created either by passing in a CSS selector, which selects all elements in the DOM that match it, or by passing in an existing array of DOM elements. Selections are the core of D3. Data can be bound to a selection, which maps the data to the DOM elements. When data is added to the selection, corresponding elements can be created and appended to a selection. When data is removed, corresponding elements will be removed from the DOM. Attributes, text, and style can all be set on a selection and updated when the data changes. A selection can be made on another selection to get child elements from the parent selection.

##### `select`

Select a single element by calling `select`. It takes a string which is a descriptor describing which element to select (think `querySelector`).

```js
import { select } from 'd3-selection';

const body = select('body');
```

##### `selectAll`

Select multiple elements with `selectAll`. It takes a string which is a descriptor of what elements to select (think `querySelectorAll`).

```js
import { selectAll } from 'd3-selection';

const links = selectAll('a');
```

A selection can be made on another selection.

```js
// select all divs in the #main element
const divs = d3.select('#main').selectAll('div');
```

## Binding Data

An element in a selection needs to have data bound to it in order to properly render. D3 provides two data binding methods: `data` and `datum`.


##### `data`

Use `data` to bind a data array to a multi-element selection (one made with `selectAll`). Its first argument is the array of data.

```js
const data = ['red', 'green', 'blue', 'yellow', 'orange'];
const selection = select('#holder').selectAll('div.element')
  .data(data);
```

D3 uses keys to map a datum to an element. By default the key is just the position in the array for a datum, so the 0th datum will be bound to the 0th element in the selection. `data` can take a second argument, which is a function that returns the key for a datum. This is useful for when your data will be updated and the ordering of datum may not be preserved.

```js
const data = [
  {color: 'red', id: 7},
  {color: 'green', id: 2},
  {color: 'blue', id: 3},
  {color: 'yellow', id: 5},
  {color: 'orange', id: 6}
];
const selection = select('#holder').selectAll('div.element')
  .data(data, d => d.id);
```

##### `datum`

Whereas `data` is used for a data array, `datum` is used to bind a single datum to a single element.

```js
const root = select('#root')
  .datum({name: 'Root', data: [1,2,3]});
```

A child selection can use its parent's data to bind data for its elements.

```js
const root = select('#root')
  .datum({name: 'Root', data: [1,2,3]});

const numbers = root.selectAll('div.number')
  .data(d => d.data);
```

## DOM Interaction

D3 provides a few methods for adding and removing elements from the DOM.


##### `append`

Given a selection, D3 appends an element with the provided tagname to each element in the selection

```js
// append an svg to the body
body.append('svg');

// append an image to each link
const imgs = links.append('img');
```

##### `insert`

Inserts a new element before the element matched by the second argument

 ```js
// insert an svg before the first element in the page
body.insert('svg', ':first-child');
```

##### `remove`

Remove all elements in the selection from the DOM

```js
links.remove();
```

When an element is appended/inserted, it is able to access the datum for its nearest bound parent element.

```js
const root = select('main')
  .datum({name: 'Root', data: [1,2,3]});

const numbers = root.selectAll('div.number')
    .data(d => d.data)
  .enter().append('div')
    .classed('number', true)
    .text(d => d)

numbers.append('span')
  .text(d => ` times two is ${d*2}`)
```

## Setting Values

**Note:** All of the below functions are described by their way to set values on selections. They can also be called without the last argument to return a value from the selection (eg `selection.attr('class')` will return the class string), but it will only return the first non-null value of the attribute/property/style from the elements in the selection, so its usefulness is limited.

##### `attr`

Sets an attribute on each element in the selection. These are HTML element attributes such as `src`, `href`, and `id`.

```js
imgs.attr('src', 'doesnotexist.jpg');
```

##### `property`

Sets the property on each element in the selection. For most attributes, `attr` is used, but occasionally you will have to use property if the HTML element doesn't support 

```js
body.append('input')
  .property('value', 'initial value');
```

##### `style`

Sets a css style for each element in the selection

```js
links.style('text-decoration', 'underline');
```

##### `classed`

The first argument is a string of classes. The second is a boolean or a function. When `true`, all of the classes listed in the first argument are added to the element. When `false`, all of the classes listed in the first argument are removed from the element. When the second argument is a function, it is called for each element in the selection to determine whether or not the class(es) should be added to the element.

```js
body.classed('foo', true);
```

##### `text`

Set the textContent of each element in the selection
 
```js
links.text('I am a link');
```

##### `html`

The innerHTML of each element in the selection

```js
links.html('<em>Italic</em> words look <strong>fancy</strong>');
```

## Enter, Exit, and Update

The elements in a selection can be in one of three states: entering, existing, or updating.

* **entering** is when a datum is bound to the selection and has no corresponding DOM element.
* **exiting** is when a DOM element no longer has a datum bound to it.
* **updating** is when a DOM element continues to have a datum bound to it after a selection has had new data bound to it.

When data is initially bound to a selection, the elements do not exist in the DOM, so they are in an entering state. D3 selections provide an `enter` method to get these elements. For each element in the entering selection, a DOM element should be appended to or inserted in its parent element.

```js
const startNumbers = [1, 2, 3, 4, 5];
let numbers = select('main').selectAll('div.number')
    .data(startNumbers)
  .enter().append('div')
    .classed('number', true)
    .text(d => d);
```

When new data is bound to a selection, some of the elements in the selection may no longer have data bound to them. These are accessible through the selection's `exit` method and the elements in it should be removed from the DOM.

```js
const oddNumbers = startNumbers.filter(n => n%2 !== 0)
numbers = numbers.data(oddNumbers)

numbers.exit().remove();
```

Elements that continue to exist after a data bind can be accessed directly by interacting with the selection.

```js
const oddNumbers = startNumbers.filter(n => n%2 !== 0)
numbers = numbers.data(oddNumbers)
  .text(d => d);
```

Sometimes when new data is bound, you may want to interact with both the entering and updating elements. D3 selections have a `merge` method to do this. It takes a second selection as its argument and merges them together. Before calling `merge`, you should perform any manipulations you want done on only entering or updating elements.

```js
const startNumbers = [1, 2, 3, 4, 5];
let numbers = d3.select('main').selectAll('div.number')
    .data(startNumbers)
  .enter().append('div')
    .classed('number', true)
    .text(d => d);

const otherNumbers = [1, 3, 5, 7, 9, 11];
// bind new data
numbers = numbers.data(otherNumbers);
// remove any elements that no longer have bound data
numbers.exit().remove();
// update any updating elements
numbers.classed('number old', true);
// add any new elements through the enter selection
numbers.enter().append('div')
  .classed('number', true)
  // then merge it with the update selection
  .merge(numbers)
  // and peform any manipulations that should be done on
  // both entering and updating elements
  .text(d => d);
```

## Other Selection Methods

D3 selections have a couple of other methods that can be used to interact with a selection.

##### `each`

calls a function on each element in the selection

```js
selectAll('a')
  // d is the datum for the current element
  // i is the index of the element in the selection
  // this is the html element
  .each(function(d, i){
    this.textContent = d + ' is the ' + i + ' element';
  });
```

##### `call`

Call a function where the argument is the selection. The difference between using `call` and directly calling the method is that the return value when using `call` is the selection.

```js
function logLength(selection) {
  console.log(selection.length);
  return true;
}

selectAll('a').call(logLength);
// this is roughly the same as calling logLength directly, but it
// returns the selection instead of the return value of logLength
logLength(d3.selectAll('a'));
```

## Transitions

Transitions are called on selections. They are provided through the `d3-transitions` module. They start after a delay (default 0) and run for a set amount of time (default 250ms). Any values set inside of a transition will be constantly updated throughout the duration of the transition, creating an animation. The values set for attributes during the transition are determined used interpolates.

### Basic Transition

```js
select('svg').selectAll('circle')
  // create a transition
  .transition()
    // set the delay before the transition starts
    // this can either be a constant or a function
    .delay(100)
    // set the duration of the transition
    // this can either be a constant or a function
    .duration(1000)
    // set the type of ease, argument is either a string
    // used to generate a function based on built-ins
    // or a function to be used
    .ease('linear')
    // transition attrs/properties/styles
    .attr('r', '50');
```

When using the `attr` method of the transition, the interpolator is automatically selected by the transition. If you want to specify which interpolator should be used, use the `attrTween` method. The second argument to that method takes a function that returns an interpolator.

`style` and `styleTween` methods also exist on a transition for altering style attributes.

```js
select('svg').selectAll('circle')
  .transition()
    .styleTween('stroke-width', function() {
      return d3.interpolateNumber(1, 3);
    })
```

### Updating Data Transitions

When data enters, updates, and exits, you can use transitions to smoothly update your visualization.

```js
// when elements exit, fade them out before removing
selection.exit()
  .transition()
    .style('opacity', 0)
    .remove();
// when elements update, transition the font size
selection
  .transition()
    .style('font-size', '32px');

// fade in elements when they enter
selection.enter().append('p')
  .text('I\'m new!')
  .style('opacity', 0)
  .transition()
    .style('opacity', 1);
```