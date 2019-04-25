# D3 Scales

A scale is a function used to map data from an input domain to an output range. When making a chart with D3, you will often need to convert values from the data to a pixel location or length within an SVG or canvas. D3 provides scale functions to handle this conversion.

**Note:** The terms 'continuous' and 'discrete' are used often to describe domains and ranges. Continuous means the set of all numbers (in a range) while discrete means only a subset of values. For example, the continuous set of numbers between 0 and 100 includes 0.1, 1, 2, 3, 3.14159, 99.999999999999, and any other number that is larger than 0 and less than 100. A discrete set of values could be the whole numbers between 0 and 100: 1, 2, 3 through to 97, 98, and 99.

## Using Scales

D3's scale functions are located in the `d3-scale` module. The official documentation for the <a href="https://github.com/d3/d3-scale">d3-scale</a> module is very good.

```bash
npm install d3-scale
```

The most common use case for scales is in setting attributes on an element. This can either be in D3's selections or in the element creation/modification/removal stages of some other code.

```js
import { scaleLinear } from 'd3-scale';

const xScale = d3.scaleLinear();
circles.attr('cx', function(d) {
  return xScale(d.x);
});
```

Scales are also used in creating axes for charts.

```js
import { axisBottom } from 'd3-axis';

const xScale = scaleLinear();
const axis = axisBottom(xScale);
```

## The Continuous Scales

A continuous scale has both a continuous domain and range.

### Linear Scales

A linear scale takes input from a continuous domain and maps it to a continuous range.

```js
import { scaleLinear } from 'd3-scale';
const linear = scaleLinear()
  .domain([0, 200])
  .range([0, 100]);

const mid = linear(100);
// mid === 50;
```

You can call the scale's `invert` method to map from the range to the domain.

```js
const inverted = linear.invert(50);
// inverted === 100;
```

A scale's domain can take more than two values in the domain, but will need a matching number of values in the range. The example below uses `interpolateRgb` to convert from numbers to colors.

```js
const polylinear = scaleLinear()
  .domain([-100, 0, 100])
  .range(['red', 'white', 'green']);
```

### Power Scales

Transform data before mapping to a range by raising the value to the scale's exponent (d^exponent).

```js
import { scaleSqrt } from 'd3-scale';

// sqrt is shorthand for scalePow().exponent(0.5)
const sqrt = scaleSqrt()
  .domain([0, 10])
  .range([0, 100]);
// the square root of 10 (~3.16) will map to 100
// the square root of 9 (3) is ~94.9% of the square root of 10
// so it will map to ~94.9
```

Set the the exponent using the exponent function (default `exponent = 1`, which is simply a linear scale)

```js
import { scalePow } from 'd3-scale';

const power = scalePow()
  .exponent(2);
const twoSquared = power(2); // twoSquared === 4
```

### Log Scales

Apply a logarithmic transformation to input data before mapping to range. The example below creates a logarithmic scale with base 10. (Reminder: `y = base^x & x = log base (y)`)

```js
import { scaleLog } from 'd3-scale';

const log = scaleLog();
const log10 = log(10); // log10 === 1
```

The base value can be modified by passing a new value to the scale's `base` method.

```js
const eLog = d3.scaleLog().base(Math.E);
```

## The Quantize Scales

Quantize scales use a discrete range and a continuous domain. Range mapping is done by dividing the domain domain evenly by the number of elements in the range. Because the range is discrete, the values do not have to be numbers.

```js
import { scaleQuantize } from 'd3-scale';

const quantize = scaleQuantize()
  .domain([0, 10])
  .range(['small', 'medium', 'long']);
const halfway = quantize(5); // halfway === 'medium'
```

Instead of being able to call invert to map from the range to the domain, call invertExent to get the upper and lower limit of domain values that will map to that range value

```js
const extent = quantize.invertExtent('medium');
// extent === [3.33333, 6.666666]
// means that any value less than 3.33333 will be 'small',
// any value between 3.33333 and 6.66666 will be 'medium',
// and all values greater than 6.66666 will be 'long'
```

## The Quantile Scales

Quantile scales are similar to quantize scales, but instead of evenly dividing the domain, they determine threshold values based on the domain that are used as the cutoffs between values in the range.

### Quantile Scales

Quantile scales take an array of values for a domain (not just a lower and upper limit) and maps range to be an even distribution over the input domain

```js
import { scaleQuantile } from 'd3-scale';

const quantile = scaleQuantile()
  .domain([1,1,1,3,4,5,6,8,12])
  .range(['red', 'white', 'blue']);

// the quantiles method returns the thresholds
// for the ranges
quantile.quantiles() // == [2.333333333333333, 5.333333333333333]
```

### Threshold Scales

Threshold scales are similar to quantile scales, but instead of determining the threshold values using an array of values, the thresholds are passed as the domain.

```js
import { scaleThreshold } from 'd3-scale';

const threshold = scaleThreshold()
  .domain([0.025, 0.16, 0.84, 0.975])
  .range(['lowest', 'low', 'average', 'high', 'highest']);
```

## The Time Scales

Time scales are similar to linear scales, but use Dates instead of numbers.

```js
import { scaleTime } from 'd3-scale';
const time = scaleTime()
  .domain([new Date('1910-1-1'), (new Date('1920-1-1'))]);

// for UTC
const utc = d3.scaleUtc();
```


## The Ordinal Scales

Ordinal scales have a discrete domain, such as a set of names.

### Ordinal Scales

Ordinal scales use a discrete range. Specifying a range will match domain values to the range using the domain value's position in the array. If the position of a domain value is larger than the number of values in the range, modular arithmetic is used to determine the range value.

```js
import { scaleOrdinal } from 'd3-scale';

const names = ['George', 'John', 'Thomas', 'James'];
const presidents = scaleOrdinal()
  .domain(names)
  .range([1, 2]);
const values = names.map(function(d){
  return presidents(d);
});
// values === [1,2,1,2]
```

An `unknown` value can be specified as the return value when an input value to the scale does not exist in the domain.

```js
presidents = presidents.unknown('???');
// presidents('Foo') === '???'
```

### Band Scales

Band scales are similar to ordinal scales, but use a continuous range instead of a discrete range. Each value in the domain will be mapped to an even length of the range, consisting of a length of band and a length of padding. This length is referred to as a step. Band scales are ideal for drawing bar charts.

```js
import { scaleBand } from 'd3-scale';

const names = ['George', 'John', 'Thomas', 'James'];
const presidents = d3.scaleBand()
  .domain(names)
  .range([0,100]);
```

The `bandwidth` method returns how long each band is. The `step` method returns the step length. Where there is no padding, these two values are equal.

```js
presidents.bandwidth() // === 25
presidents.step() // === 25
```

The step length is not always a whole number, meaning that bands won't map perfectly to pixels. You can specify that the steps should be rounded to the nearest pixel using either the `round` method or using `rangeRound` instead of `range`

```js
presidents.round(true);
// or
presidents.rangeRound([0, 100]);
```

Padding can be added, so that each band does not rub against its neighbors. Inner padding is placed between each band. Outer padding is added before the first and after the last band. How much of the outer padding is before the first band and how much is after the last band is determined by the align value, which defaults to 0.5 (splitting the padding evenly before and after). Inner padding will shorten the length of each band, but does not directly affect step length. Outer padding affects step length by reserving a portion of the total length of the range as padding.

```js
presidents
  // the inner padding is 10% of the step length
  .paddingInner(0.1)
  // 100% of the length of a step (0.5*2) should
  // be used as padding before and after the first
  // and last steps
  .outerPadding(0.5)
  // 25% of the outer padding length should be
  // before the first step and 75% should be after
  // the last step
  .align(0.25);
```

The padding can also be set using the `padding` method, which will set both the inner and outer padding to the provided value.

```js
presidents.padding(0.25);
```

### Point Scales

Point scales are band scales with no band width. While band scales are useful for bar charts, point scales are more useful for scatterplots.

```js
import { scalePoint } from 'd3-scale';

const years = ['1796', '1800', '1804', '1808', '1812'];
const electionYears = scalePoint()
  .domain(years);
```

Because a point scale has no band width, it has no need for inner padding. It has no `innerPadding` and `outerPadding` methods. Instead, the outer padding is specified with the `padding` method.

## Additional Scale Methods

Below are some more methods that exist for scales. Not every scale has all of the methods listed below. Check out the source code or <a href="https://github.com/d3/d3-scale#">documentation</a> to see which scales use them.


### Clamp

Clamp forces output values to be within the range, regardless of whether the input values are in the domain.

```js
const linear = scaleLinear();
// linear.range() === [0,1]
// linear.domain() === [0,1]

const outside = linear(2);
// outside === 2

// turn clamping on
linear.clamp(true);
const inside = linear(2);
// inside === 1
```

### Interpolate

You can specify a different interpolator function using the `interpolate` method. This lets you create a scale that returns something other than a number.

```js
const percent = scaleLinear()
  .domain([0, 100])
  .interpolate(d3.interpolateString)
  .range(['0% done', '100% done']);

const halfway = percent(50);
// halfway === '50% done'
```

### Nice

The `nice` method rounds the domain's extent values to nicer looking numbers.

### Copy

The `copy` method duplicates a scale. Most scale methods modify a scale. Any methods called on the duplicated scale will not affect the original scale, and vice versa.

### Tick Methods

When creating a chart using D3 (a common use case for scales), you will probably find yourself wanting to draw an axis or two. Scales provide methods that are useful in creating the ticks that are drawn on axes.

##### Ticks

Ticks returns an array of tick values for a scale. The array will be evenly distributed values within the domain. You can provide a value to the function to specify how many ticks you want. The returned array's length should be approximately how many ticks are requested, but may differ slightly depending on how well the data can be spaced within the domain.

```js
const scale = scaleLinear()
  .domain([0, 10])
  .range([0, 100]);

// scale.ticks() === [0,1,2,3,4,5,6,7,8,9,10]
// scale.ticks(5) === [0,2,4,6,8,10]
```

When using a time scale, you can also pass in a time interval instead of a tick count.

```js
const time = scaleTime()
  .domain([new Date(2010, 0, 1), new Date(2010, 11, 31)]);

const ticks = time.ticks(d3.timeWeek);
```

##### Tick Format

The `tickFormat` method returns a function that will format tick values to strings for output. It takes two arguments, the first of which is a count and should be the same as the value you pass to the `ticks` method. The second (which is optional) is a formatting string. This is useful when tick values should be rounded or when you want to add additional information to them (like a percent or dollar sign).

```js
const moneyScale = scaleLinear()
  .domain([0, 100])
  .range([0, 1000]);

const format = moneyScale.tickFormat(5, '$.2f');
moneyScale.ticks(5).forEach(function(t) {
  console.log(format(t));
});
