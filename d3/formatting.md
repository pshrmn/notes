# D3 Formatting

When you want to output values in your data as strings, D3's formatting functions are your friends. These help you to control how numbers and dates appear. With formatting functions, you no longer have to worry about your numbers output with a dozen decimal places.

## Usage

D3's formatting functions are the in `d3-format` and `d3-time-format` modules.

```bash
npm install d3-format d3-time-format
```

## Formatting Strings

`format` takes a specifier string and returns a function that will format numbers. Below are a few examples of useful formatting string specifiers. For more specific formatting rules, check out the `d3-format` <a href="https://github.com/d3/d3-format#locale_format">documentation</a>.

```js
import { format } from 'd3-format';
```

#### Thousands separator

```js
// Use a comma as a thousands separator
const commas = format(',');
const millions = commas(1234567);
// millions = '1,234,567'
```

#### Positive/Negative

```js
// Use a plus sign to show a +/- in front of positive/negative numbers
const signed = format('+');
const pos = signed(100);
// pos = '+100'

const neg = signed(-23);
// neg = '-23'
```

#### Fixed number of digits

```js
// Number after period for fixed length mantissa
const twoCharacters = format('2f');
const oneTwo = twoCharacters(1);
// oneTwo === ' 1'

const threeDigits = format('.3f');
const pi = threeDigits(3.14159);
// pi === '3.142'
```

#### String length

```js
// Number before period specifies how many characters the returned string should be
const five = format('5f');
const output = five(23);
// output.length = 5
// output === '   23'
```

#### Integer

```js
// Use a zero after the decimal point to specify that output should
// be an integer. This will round the number up or down.
const integer = format('.0f');
const pi = integer(Math.PI);
// pi === '3'
```

#### Round

```js
import { precisionRound } from 'd3-format';
// precisionRound rounds a number to the given number of decimal places.

const pi2 = d3.precisionRound(3.14159, 2);
// pi2 === '3.14'

// round defaults to 0 decimal places
const pi3 = d3.precisionRound(3.14159);
// pi3 === 3
```

## Formatting Time

**Note:** For more specific time formatting rules, check out the `d3-time-format` <a href="https://github.com/d3/d3-time-format#locale_format">documentation</a>.

```js
import { timeFormat } from 'd3-time-format';
const date = new Date(1950, 0, 17);
// %Y - return year in format XXXX
const year = timeFormat('%Y');
const y = year(date);
// y === '1950'
```


#### parse

A time formatter can work in reverse to return a date

```js
const year = timeFormat('%Y');
const newYears = year.parse('1990');
// newYears returns the same value as call to new Date(1990, 0, 1)
```
