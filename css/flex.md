# Flexbox

https://css-tricks.com/snippets/css/a-guide-to-flexbox/

CSS rules for flexbox are divided between container properties (the element with `display: flex;`) and item properties (the direct descendant elements of the container).

## Container Rules

### display: flex;

Sets an element as a flexbox.

This can also be `inline-flex` for inline containers.

### flex-direction

`row` (`row-reverse`) or `column` (`column-reverse`).

### flex-wrap

`nowrap` (default), `wrap`, or `wrap-reverse`.

### flex-flow

A combination of `flex-direction` and `flex-wrap`.

### justify-content

Alignment of items along the flex direction axis. For `flex-direction: row`, this will align items along the x-axis.

`flex-start` - content starts at beginning of axis
`flex-end` - content ends at end of axis
`center` - center content
`space-between` - first item at beginning of axis, last item at end of axis, items spaced evenly
`space-around` - each item has the same spacing before and after it; this means that the space before the first item and after the last item is half of the space between adjacent items
`space-evenly` - even spacing around all items (including before first and after last items)

### align-items

Alignment of items and the axis opposite of the flex direction. For `flex-direction: row`, this will align items along the y-axis.

`flex-start` - items begin at beginning of axis
`flex-end` - items end at end of axis
`center` - items centered on center of axis
`stretch` - items stretch to fill full axis (unless other properties specify a more specific size)
`baseline` - items aligned along their baselines

### align-content

Controls alignment of wrapped lines

`flex-start`
`flex-end`
`center`
`space-between`
`space-around`
`stretch`

## Items

### order

Control the order that the item is displayed in. Defaults to `0` but can be a positive or negative integer.

### flex-grow

Control how the item expand along its `flex-direction` axis. Each item in a line will contribute to the determined size. The default value `order` is 0 and the order must be positive.

If there are three items, the first with `order: 1;`, the second with `order: 2;` and the third with `order: 3;`, the third's width will be 50% (3/6), the second will be 33% (2/6), and the first will be ~17% (1/6).

Order accumulates on top of hard-coded sizing, like a `width`.

### flex-shrink

Control how an item shrinks to fit into a line. This defaults to `1`.

### flex-basis

Define the base size of an item (e.g. `50px` or `5em`).

If this is `auto`, the value will be determined by the `width` or `height` (depending on its container's `flex-direction`).

### flex

A combination of `flex-grow`, `flex-shrink`, and `flex-basis`. Defaults to `0 1 auto`.

Only one argument is required; which rule this specifies is determined by the input.

```css
flex: 50px; /* flex-basis */
flex: 1; /* flex-grow */
flex: 50px 3; /* flex-basis, flex-grow */
flex: 3 4; /* flex-grow, flex-shrink */
```

### align-self

Controls the items alignment on the axis opposite the `flex-direction`; this overrides the `align-items` value.

## Resources

* https://mastery.games/p/flexbox-zombies
