# Interpolators

## About Interpolators

An interpolator takes two arguments, a lower and an upper bounds. These two values are mapped to `0` and `1`. Given a value between and including `0` and `1`, which represents a percentage from 0% to 100%, the interpolator will return its equivalent value between the lower and upper bounds.

Interpolators are an important part of transitions. A transition on an attribute starts with one value, is constantly updated over a set period of time, and then is given an end value. At each update time, the percent of time that has elapsed is used to determine what percent of the way through the transition we are currently at. That value is passed to an interpolator to determine what the value the attribute should have right now.

## Using Interpolators

D3's interpolators are in the `d3-interpolate` module.

```bash
npm install d3-interpolate
```

## D3 Interpolators

D3 has a number of interpolators built-in.

### Generic Interpolator

```js
import { interpolate } from 'd3-interpolate';
const interpolator = interpolate(0, 100);
```

`interpolate` returns the correct interpolator based on the second argument. In order, it will make the following checks on the value of the second argument:


* `null`, `undefined`, `true`, or `false`? The interpolator function will always return the second value.
* Number? `interpolateNumber`
* Color? `interpolateRgb`
* Date? `interpolateDate`
* String? `interpolateString`
* Array? `interpolateArray`
* Can coerce to a number? `interpolateNumber`
* Everything else failed? `interpolateObject`

### Interpolate Numbers

`interpolateNumber` is used to interpolate between two numbers.

```js
import { interpolateNumber } from 'd3-interpolate';
const nums = interpolateNumber(0, 10);
const pi = nums(0.314);
// pi === 3.14
```

It is better to interpolate from 1e-6 than from 0 when the result is used as a string to prevent an interpolated number from appearing in scientific notation.

### Interpolate Colors

`interpolateRgb` takes two colors as its arguments. The colors may either be css colors or rgb hex strings of either 3 digits (eg '#123') or 6 digits (eg '#123DEF').

```js
import { interpolateRgb } from 'd3-interpolate';
const colors = interpolateRgb('red', '#00FFFF');
```

A hash table is used to lookup the rgb value of known css colors.

```js
import { range } from 'd3-array';

const weirdColors = interpolateRgb('chartreuse', 'oldlace');
const weirds = range(2).map(weirdColors);
// weirdColors(0) === '#7fff00'
// weirdColors(1) === '#fdf5e6''
```

If a value passed to the interpolator is not a valid color string of rgb hex string, it defaults to #000000.

```js
const unknownColor = interpolateRgb('red','some fake color');
const unknown = unknownColor(1);
// unknown === '#000000'
```


Colors can also be in HSL (`interpolateHsl`), L*a*b* (`interpolateLab`), or HCL (`interpolateHcl`), but will return an rgb hexadecimal string
```js
import { interpolateHsl } from 'd3-interpolate';
const hsl = interpolateHsl('hsl(0, 100%, 50%)',' hsl(255, 100%, 50%)')
const halfHsl = hsl(0.5)
// halfHsl === '#ff00df'
```

### Interpolate Dates

`interpolateDate` is used to interpolate between two `Date` objects.

```js
import { interpolateDate } from 'd3-interpolate';
const year = interpolateDate(new Date('01-01-2016'), new Date('01-01-2017'));
```

### Interpolate Strings

`interpolateString` finds any numbers in the second argument, and attempts to match it to a number in the first argument. Non-number parts and unmatched numbers in the second argument string are used statically as a template.

```js
import { interpolateString } from 'd3-interpolate';
const strings = interpolateString('I am 0% complete', 'I am 100% complete');
const halfway = strings(0.5);
// halfway === 'I am 50% complete'
```

In the above example, only the 0 is needed in the first string.

```js
const partialStrings = d3.interpolateString('0', 'I am 100% complete');
const partString = partialStrings(0.5);
// partString === 'I am 50% complete'
```

If there are more numbers in the second string than the first, unmatched numbers will be included as part of the template.

```js
const unmatched = d3.interpolateString('0', 'it is 15 minutes until 7 o\'clock');
const halfmatched = unmatched(0.5);
// halfmatched === 'it is 7.5 minutes until 7 o'clock'
```


### Interpolate Array

`interpolateArray` matches values in the second array to corresponding values in the first. For each value in the array with a corresponding one in the first, a generic interpolator is created. If there is no matching value in the first array, the value is used statically in the template.

Interpolate array matches based on values in the second array.

```js
import { interpolateArray } from 'd3-interpolate';
const arrays = interpolateArray([1,2,3],[4,5,6]);
const third = arrays(1/3);
// third === [2,3,4]
```

If there is no matching element in the first array to one in the second, that value is used statically

```js
const nomatch = interpolateArray([1,2],[3,4,5]);
const halfway = nomatch(0.5);
// halfway = [2,3,5];
```

Any type of nested elements in the array will work

```js
const anytype = d3.interpolateArray([0, {x:10}], ['testing 10', {x:100}]);
const mixedResults = anytype(0.5);
// mixedResults = ['testing 5', {x:55}]
```

### Interpolate Object

`interpolateObject` uses all of the properties in the second argument, matching to properties in the first argument.

```js
import { interpolateObject } from 'd3-interpolate';
const obj = interpolateObject({x:0, y: 1}, {x: 50, y: 25});
const halfway = obj(0.5);
// halfway === {x: 25, y: 13}
```

Unmatched properties are used as static templates.

```js
const staticObj = interpolateObject({x: 0}, {x: 10, y: 7});
const quarter = staticObj(0.25);
// quarter === {x: 2.5, y: 7}
```

Properties of the first argument that don't exist in the second argument are also used as a static template

```js
const ignoreObj = interpolateObject({x: 0, y: 10}, {x: 100});
const threeQuarters = ignoreObj(0.75);
// threeQuarters === {x: 75, y: 10}
```
