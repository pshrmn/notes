# Big List of AST Node Types

These are Acorn node types, since that is what Rollup uses.

## Types

### ArrayPattern

#### Example 

```js
// the array param
function thing([a, b, c]) {}
```

#### Properties

* elements - array of `Identifier`s and `AssignmentPattern`s

### ArrowFunctionExpression

##### Example 

```js
() => {}
```

##### Properties

* params - array of `Identifier`s or patterns
* body - `BlockStatement`, expression, or `Literal`

### AssignmentExpression

#### Example 

```js
x = 5
```

#### Properties

* left - `Identifier` or pattern
* right - a `Literal` or expression
* operator - `=`, `+=`, etc.

### BinaryExpression

#### Example 

```js
variable === 24
```

#### Properties

* left - `Literal` or expression
* right - `Literal` or expression
* operator - `==`, `+`, etc.

### BlockStatement

#### Example 

```js
{
  const x = 'hello';
  console.log(x);
}
```

#### Properties

* body - array of statements and declarations

### CallExpression

#### Example 

```js
fn(one, two)
```

#### Properties

* callee - `Identifier` or `MemberExpression`
* arguments - array of `Literal`s and expressions

### CatchClause

#### Example 

```js
catch (e) {
  ...
}
```

#### Properties

* param - an `Identifier`
* body - a `BlockStatement`

### ClassDeclaration

#### Example 

```js
class Thing {}
class AnotherThing extends Thing {}
// the class part of:
export default class {}
```

#### Properties

* id - an `Identifier`
* superClass - an `Identifier` or null
* body - a `ClassBody`

### ClassExpression

#### Example 

```js
// the right node of
var x = class Thing {};
```

#### Properties

* id - an `Identifier` (optional)
* superClass - an `Identifier` or null
* body - a `ClassBody` 

###  ConditionalExpression

#### Example 

```js
test ? consequent : alternate
```

#### Properties

* test - an expression or `Identifier`
* consequent - an expression or `Literal`
* alternate - an expression or `Literal`

### EmptyStatement

#### Example 

```js
;
```

### ExportAllDeclaration

#### Example 

```js
export * from 'another-module';
```

#### Properties

* source - a `Literal` string

### ExportDefaultDeclaration

#### Example 

```js
export default function() {...}
```

#### Properties

* declaration

### ExportNamedDeclaration

#### Example 

```js
export const x = 'Exit';  // declaration

const y = 'Why';
export { y }              // specifiers
```

#### Properties

* declaration
* specifiers - an array of export specifiers

**Note:** This is a one or the other type of situation

### ExpressionStatement

#### Example 

```js
variable = value;
fn();
```

#### Properties

* expression

### ForInStatement

#### Example 

```js
for (let key in obj) {
  ...
}
```

#### Properties

* left - a `VariableDeclaration`
* right - an `Identifier` or expression
* body - a `BlockStatement`

### ForOfStatement

#### Example 

```js
for (let key of iterable) {
  ...
}
```

#### Properties

* left - a `VariableDeclaration`
* right - an `Identifier` or expression
* body - a `BlockStatement`

### ForStatement

#### Example 

```js
for (init; test; update) {
  ...
}
```

#### Properties

* init - a `VariableDeclaration` or null
* test - an expression
* update - an expression
* body - a `BlockStatement` or expression

### FunctionDeclaration

#### Example 

```js
function name(param1, param2) {
  ...
}
```

#### Properties
* id - `Identifier`
* generator - boolean
* expression - boolean (???)
* params - an array of `Identifier`s or patterns
* body - a `BlockStatement`

### FunctionExpression

#### Example 

```js
// the right part of:
let fn = function() {}
```

#### Properties

* id - `Identifier` or null
* generator - boolean
* expression - boolean (???)
* params - an array of `Identifier`s or patterns
* body - a `BlockStatement`

### Identifier

#### Example 

```js
// x in:
const x = 'hello';
```

#### Properties

* name - a string

### IfStatement

#### Example 

```js
if (condition) {
  ...
} else {
  ...
}
```

#### Properties

* test - an `Identifier` or expression
* consequent - a `BlockStatement`
* alternate - An `IfStatement` or `BlockStatement`

### ImportDeclaration

#### Example 

```js
import Name from 'module';
import { One, Two } from 'module';
import * as Grouped from 'module';
```

#### Properties

* specifiers - array of specifiers
* source - `Literal`

### Literal

#### Example 

```js
// the string in
const x = 'test';
```

#### Properties

* value - the literal value
* raw - the value as a string

### LogicalExpression

#### Example 

```js
thing && thing.property
```

#### Properties

* left - an `Identifier`, `Literal`, or expression
* operator - '&&' or '||'
* right - an `Identifier`, `Literal`, or expression

### MemberExpression

#### Example 

```js
object.property
object['property']
```

#### Properties

* object - an `Identifier` or expression
* property - an `Identifier` or `Literal`
* computed - boolean

### NewExpression

#### Example 

```js
new Thing(arg1, arg2)
```

#### Properties

* callee - `Identifier` or expression
* arguments - an arry of `Identifier`s or expressions

### ObjectPattern

#### Example 

```js
{ one: 1, two: 2 }
```

#### Properties

* properties - an array of `Property` nodes

### Property

#### Example

```
// any of the lines below (a,b,c, d)
{
  a: 'Ayy',
  b() {...},
  c,
  [d]: 'Dee'
}
```

#### Properties

* method - true if if `key()` format
* shorthand - true if in `{ key }` format
* computed - true if in `{ [key]: ... }` format
* key - an `Identifier`
* value - a `Literal`, pattern, or expression
* kind - "init"??

### RestElement

#### Example

```js
// the "...rest" element
const [first, ...rest] = [1,2,3,4];
```

#### Properties

* argument - an `Identifier`

### ReturnStatement 

#### Example

```js
return something;
```

#### Properties

* argument - an `Identifier`, `Literal`, or expression

### SwitchStatement

#### Example 

```js
switch (discriminant) {
case 1:
  ...
  break;
case 2:
  ...
  break;
default:
  ...
  break;
}
```

#### Properties

* discriminant - an `Identifier` or expression
* cases - an array of `SwitchCase` nodes

### TaggedTemplateExpression

#### Example 

```js
tag\`template string\`
```

#### Properties

* tag - an `Identifier` (possibly a function expression?)
* quasi - a `TemplateLiteral`

### TemplateLiteral

#### Example 

```js
`I am a template literal`
`Name: ${obj.name}\nAge: ${obj.age}`
```

#### Properties

* expressions - an array of `Identifier`s or expressions
* quasis - an array of `TemplateElement`s

### ThisExpression

#### Example 

```js
// just the "this" part of the following lines
this.value = arg => { console.log(arg); };
this.value('hi!');
```

### ThrowStatement

#### Example 

```js
throw new Error();
```

#### Properties

* argument - an `Identifier` or expression

### UnaryExpression

#### Example 

```js
+plus
!not
~flipBits
typeof joke
```

#### Properties

* operator - `+`, `-`, `!`, `~`, `typeof`, `void`, `delete`
* prefix - true
* argument - an `Identifier` or expression

### UpdateExpression

#### Example 

```js
variable++
```

#### Properties

* operator - '++' or '--'
* prefix - false
* argument - an `Identifier` or expression

### VariableDeclaration

#### Example 

```js
var one = 1;
let two = 2;
const three = 3, four = 4;
```

#### Properties

* kind - 'var', 'let', or 'const'
* declarations - an array of `VariableDeclarator`s

### VariableDeclarator

#### Example 

```js
x = 'exit'
{ y, z } = obj
[ a, b ] = arr
```

#### Properties

* id - an `Identifier` or pattern
* init - a `Literal`, `Identifier`, or expression