# Jest + Babel

## `babel-jest`

With Babel 7, modules in `node_modules/` will no longer "inherit" the Babel config from the source of the `node_modules/`.

In the below example, if `pkg/module.js` needs to be transformed using `babel-jest`, it **will not** use `project/.babelrc.js`.

```
project/
  src/
    index.js
  node_modules/
    pkg/
      module.js
  .babelrc.js
```

There are two ways to "fix" this.

### Custom Transformer

Instead of passing `babel-jest` in the Jest configuration, a custom transformer can be created, to which you would pass the desired Babel configuration options.

```js
// transformer.js
const config = require("./.babelrc.js");
module.exports = require("babel-jest").createTransformer(config);
```

```js
// jest.config.js
module.exports = {
  transform: {
    "\\.js$": "./transformer"
  }
}
```

### Root Config

Instead of using a `.babelrc` or `.babelrc.js` file, you can use the `babel.config.js` file. This acts as a global configuration, so files in `node_modules/` will inherit its configuration.

```js
// babel.config.js
module.exports = {
  presets: [
    ["env", {
      modules: "cjs"
    }]
  ]
};
```
