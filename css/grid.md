# Grid

* https://css-tricks.com/snippets/css/complete-guide-grid/
* https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Basic_Concepts_of_Grid_Layout

Grid is a two dimensional layout system (as opposed to flexbox being one dimensional).

A grid is made up of horizontal and vertical lines. The spaces between these lines (where content is displayed) are known as tracks.

Each grid line is numbered, starting with index `1`. Horizontally, for left-to-right languages, 1 is the left-most, while it is the right-most for right-to-left. (Note: verify how switching languages swaps LTR/RTL). Vertically, 1 is the top line.

Items in the grid can be given an area name. This name is used to place the item in the grid.

CSS rules for grid are divided between container properties (the element with `display: grid;`) and item properties (the direct descendant elements of the container).

## Terminology

* `track` - the space between lines of the grid
* `cell` - the smallest unit of a grid. multiple items can occupy the same cell
* `area` - a rectangular shape made up of one or more grid cells
* `gutter`/`alley` - space between cells
* `fr` - a measurement unit representing one fraction of the grid's width (for columns) or height (for rows). Fractions are calculated after hard-coded widths.

## Container Rules

### display: grid

Sets an element as a grid.

This can also be an `inline-grid`.

### grid-template-columns, grid-template-rows

**Note:** The following properties apply to both `grid-template-columns` and `grid-template-rows`, but the examples will only refer to `grid-template-columns`.

A space separated list of track sizes.

A track can either use standard CSS units, `fr`s, or one of a number of special types.

```css
grid-template-columns: 100px 1fr 2fr 1fr;
/*
the first cell will be 100px, the other cells
will be their fraction out of the sum of the fractions (4)
*/
```

Line will automatically be assigned two names: a positive number (based on position) and a negative number (reverse position). The lines can be named using square brackets. A line can be given multiple names by separating them with spaces in the assignment.

```css
grid-template-columns: [one-start] 100px [one-end two-start] 100px [two-end];
```

#### Special Types

`auto` inherits min/max size.

`minmax` is used to specify minimum/maximum sizes for a track

```css
grid-template-columns: minmax(100px, 1fr) 1fr 1fr;
```

`repeat` is used to define multiple tracks that fit into a pattern

```css
grid-template-columns: repeat(1fr);
```

`auto-fill` adds as many columns (or rows) as will fit, including empty columns (or rows).

`auto-fit` adds as many columns (or rows) as will fit, then collapses empty columns (or rows) and expands filled ones.


### grid-template-areas

Define a grid template using grid area names. The areas are defined with a string per row and space separated grid area names per column.

Names can be used multiple times to span multiple cells.

An empty cell is denoted by a period (`.`).

```css
grid-template-areas:
  "one one two"
  "three three two"
  ". . four";
```

### grid-template

A combination of `grid-template-columns`, `grid-template-rows`, and `grid-template-areas`.

### gap, row-gap, column-gap

Define the gutter size between cells. `gap` can be used as shorthand; if provided two values they represent `gap: row-gap column-gap;`.

**Note:** These properties may need to be prefixed with `grid-` for older browser support.

### justify-items / align-items

`justify-items` aligns items along the row axis (left/right), while `align-items` aligns items along the column axis (top/bottom).

* `start` - starting edge of the cell
* `end` - ending edge of the cell
* `center` - center of the cell
* `stretch` - fill entire cell (default)

### place-items

A combination of `justify-items` and `align-items`.

```css
place-items: align-items justify-items;
```

**Note:** Not supported in (pre-chromium?) Edge.

### justify-content / align-content

Control horizontal/vertical alignment of the grid within the grid container (if it is smaller than the container).

These have the same properties as [`flexbox`'s `justify-content`](./flex.md#justify-content).

### place-content

A combination of `justify-content` and `align-content`.

```css
place-content: align-content justify-content;
```

### grid-auto-columns / grid-auto-rows

Control the size of implicit tracks

### grid-auto-flow

Control the placement of items that aren't explicitly placed

* `row` - fill in each row, adding rows if necessary (default)
* `column` - fill in each column, adding columns if necessary
* `row dense`/`column dense` - fill in holes where possible (probably avoid, can be bad for a11y)

### grid

Shorthand for `grid-template-rows`, `grid-template-columns`, `grid-template-areas`, `grid-auto-rows`, `grid-auto-columns`, and `grid-auto-flow`.

This seems fairly complex and easy to mess up, so I'll probably avoid it until I am more comfortable with CSS grids.

## Item Rules

### grid-column-start / grid-column-end / grid-row-start / grid-row-end

Specify an item's location using line names.

```css
grid-column-start: 1;
grid-column-end: 3;
```

`span <name>` tells the item to span across the named line instead of ending at it.

`span <number>` tells the item to span across the provided number of tracks.

### grid-column / grid-row

Shorthand for combining the start and end values.

```css
grid-column: start / end;
grid-row: start / end;
```

### grid-area

Name an item for use in `grid-template-areas` or a combination of `grid-column` and `grid-row`.

```css
grid-area: test;
grid-area: col-start / col-end / row-start / row-end;
```

### justify-self

Override value from grid container's `justify-items`.

### align-self

Override value from grid container's `align-items`.

### place-self

Combination of `justify-self` and `align-self`.

```css
place-self: align-self justify-self;
```
