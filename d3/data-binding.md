# D3 Data Binding

## Data Binding

### Data
        
An array of data can be joined to a selection by calling the `data` function on it. By default, data is bound to the selection by position in the array. In the example, the first `<p>` element in the `ps` selection will be bound to the `'David'` datum.

```js
const names = ['David', 'Susan', 'Monica', 'Gordon', 'Elizabeth', 'William'];

// create a paragraph for each name
const ps = d3.select('body').selectAll('p');
// the selection does not have enter/exit functions
// ps.enter === undefined

// use the position in the array as the key
ps.data(names);
```

While using the position is good enough for simple data binding, this can break down when new data is bound to a selection. If a new datum was added to the beginning of the data, then each element will end up being bound to a new datum. Instead, it is better to pass in a `key` function to identify datum. Then, when updating the data, elements whose key-bound datum is still in the data array will remain bound to the same datum.

```js
const names = [
  {name: 'David', age: 37},
  {name: 'Susan', age: 32},
  {name: 'Monica', age: 27}
];
const newNames = names.slice(1);

let ps = d3.select('body').selectAll('p');

// default - use the index as the key
ps = ps.data(names);
/*
* p0 - key=0, data={name: 'David', ...}
* p1 - key=1, data={name: 'Susan', ...}
* p2 - key=2, data={name: 'Monica', ...}
*/
ps = ps.data(newNames);
/*
* the first p now has to update with the Susan's data and the third p
* has to be removed
* p0 - key=0, data={name: 'Susan', ...}
* p1 - key=1, data={name: 'Monica', ...}
*/

// use the name as the key
ps = ps.data(names, function(d) {
  return d;
});
/*
* p0 - key="David", data={name: 'David', ...}
* p1 - key="Susan", data={name: 'Susan', ...}
* p2 - key="Monica", data={name: 'Monica', ...}
*/
ps = ps.data(newNames, function(d) {
  return d;
});
/*
* the first p will be removed and the second and third 'p's will keep
* their data the same
* p1 - key="Susan", data={name: 'Susan', ...}
* p2 - key="Monica", data={name: 'Monica', ...}
*/
```
      
### Enter and Exit

After the data is bound, the selection has enter and exit methods to handle the fact that the number of elements in your selection and the number of datum in your bound data might not line up. To handle new data, you call `enter()` followed by `append(element)` (where element is the type from the selection). To remove elements that no longer have data bound to them, call `exit()` followed by `remove()`.

```js
const names = [
  {name: 'David', age: 37},
  {name: 'Susan', age: 32},
  {name: 'Monica', age: 27}
];

// ps is the selection with bound data
const ps = d3.select('body').selectAll('p')
  ps.data(names)

// create 'p' elements for new data
ps.enter().append('p');

// remove elements that have no bound data
// this does nothing here, but is important for data updates
ps.exit().remove();
```

### Update

The data for a selection can be updated with a new array of data. When no key is provided, data is overriden based on array index. Call the exit function on the selection to handle elements that no longer have data bound to them (such as if the length of the new data array is shorter than the previous data array).

```js
// set new data for names
const names = ['James', 'George', 'John', 'Barack'];
let paragraphs = d3.select('body').selectAll('p.name')
  .data(names, function(d) { return d; });

// create a <p class="name"> for each datum in names
paragraphs.enter().append('p')
  .classed('name', true)
  .text(function(d){ return d; });

const newNames = ['James', 'John', 'Dwight'];
paragraphs = paragraphs.data(newNames, function(d) { return d; });

// EXIT 'George' and 'Barack'
paragraphs.exit().remove();

// ENTER 'Dwight'
paragraphs.enter().append('p')
    .classed('name', true)
  // MERGE the ENTER with the UPDATE
  .merge(paragraphs)
    .text(function(d){ return d;}); 
```

### Datum

A datum can be bound to a single element by calling the datum function. This selection does not contain enter/exit functions.

```js
const names = ['David', 'Susan', 'Monica', 'Gordon', 'Elizabeth', 'William'];

// create a paragraph for each name
d3.select('body').select('div')
  .datum(names);
```

### Nested Data

Data can be passed down from a parent element with bound data

```js
const names = ['David', 'Susan', 'Monica', 'Gordon', 'Elizabeth', 'William'];

// create a paragraph for each name
const people = d3.select('body').append('div')
  .datum(names);

people.selectAll('p')
    .data(function(d){ return d;})
  .enter().append('p')
    .text(function(d){ return d;});

// if the parent data is not an array, it should be wrapped in brackets
// to create a length one array. Otherwise it might not behave as expected.
// eg. .data('string') will have an element for each character 
// ('s', 't', 'r', 'i', 'n', and 'g') whereas .data(['string']) will have
// just one element with data 'string'
const agents = d3.select('body').append('div')
  .datum('Agent Smith');

agents.selectAll('p')
    .data(function(d){ return [d];})
  .enter().append('p')
    .text(function(d){ return d;});
```
   
## Working with Bound Data
   
Once that data has been bound and elements have been created, we have a selection. This allows us to use the selection's method (attr, text, style, property) to modify each element in the selection.

```js
// attributes can be set on selections
const peopleData = [{name: 'Joe', age: 31, 'gender': 'male'},
  {name: 'Doug', age: 42, gender: 'male'},
  {name: 'Jill', age: 37, gender: 'female'}];

const people = d3.select('body').selectAll('p.person')
    .data(peopleData, d => d.name)
  .enter().append('p')
    .classed('person', true)
    .classed('male', d => d.gender === 'male')
    .classed('female', d => d.gender === 'female')
    .text((d, i) => `Person ${i} is named ${d.name} and is ${d.age}`);
```

### Loading Data

Large datasets will most likely needed to be loaded from an external source. The `d3-request` module provides a number of functions for making these requests. The `request` function makes a generic request that you can configure to your liking. If you know the type that the response should be, there are a number of other functions that are pre-configured to make requests and parse the response for you.

```js
d3.json('path.json', funtion(error, data){
  if ( error ) {
    // handle the error
  }
  // do something with the loaded data
});

d3.text('path.txt', function(error, data){});

d3.xml('path.xml', function(error, data){});

d3.html('path.html', function(error, data){});

d3.csv('path.csv', function(error, data){});

d3.tsv('path.tsv', function(error, data){});
```

For CSV and TSV data, each row will need to be parsed. A function can be passed to those methods, as the second argument to the `d3.csv` and `d3.tsv` functions, which specifies how to parse each row. If no parse function is provided, the first row is assumed to be a header row, and for every other row in the response an object will be created. The keys in each object will be the values in the header row. The values for each key will be the item in the row at the same index as the header key.

```csv
name,age
John,31
Carol,49
```

```js
d3.csv('data.csv', function(error, data) {
  // data is the parsed csv file as an array
  // data === [{name: 'John', age: 31}, {name: 'Carol', age: 49}]
});
```