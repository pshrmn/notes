# Testing Svelte

## Jest Setup

Testing Svelte components with Jest requires you to setup a Jest transformer. This is a module that exports a `process()` function. `process()` should either return a string of code or an object with a `code` property (whose value is a string of code).

```js
// scripts/svelteCompile.js
const svelte = require("svelte");

exports.process = function process(src) {
  const result = svelte.compile(src, {
    format: "cjs"
  });
  return result.js.code;
};
```

Jest also needs to be setup to know what files to transform. This can be done by adding `html` to the known file extensions and telling Jest to use the transformer for those files. A JavaScript transformer should also be specified (e.g. `babel-jest`).

```js
// jest.config.js
module.exports = {
  moduleFileExtensions: ["js", "html"],
  transform: {
    "\\.js$": "babel-jest",
    "\\.html$": "./scripts/svelteCompile"
  },
  transformIgnorePatterns: ["node_modules/(?!(svelte)/)"]
};
```

If the app is using Svelte's store (`svelte/store`), that will also need to be compiled. Jest typically ignores files in `node_modules/`, but files can be whitelisted. `(?!(svelte)\/)` will skip (i.e. _not_ ignore) any filenames in the `svelte` package.

## Jest Testing

Testing a component is done by importing it and creating an instance. Jest comes with jsdom built-in, which makes mounting the instance simple. The (fake) DOM can then be queried and asserted against.

```html
<!-- src/MyComponent.html -->
<div>{value}</div>
```


```js
import MyComponent from "../src/MyComponent.html";

describe("<MyComponent>", () => {
  it("does something", () => {
    const node = document.createElement("div");
    const Cmp = new MyComponent({
      target: node,
      data: {
        value: "hi!"
      }
    });
    const div = document.querySelector("div");
    expect(div.textContent).toBe("hi!");
  });
});
```

### Events

[`simulant`](https://github.com/Rich-Harris/simulant) makes testing events fairly simple.

```js
const a = node.querySelector("a");
const event = simulant("click");
simulant.fire(a, event);
```
