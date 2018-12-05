# Type Overview

## Basic Types

[TypeScript Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)

The primitive types are `string`, `number` (for both ints and floats), and `boolean`.

```ts
const x: string = "hi!";
const y: number = 5;
const z: boolean = true;
```

A type can be either `undefined` or `null`. These are valid for any variable unless the `strictNullChecks` flag is used.

```ts
const a: number; // currently undefined
```

The `void` type is useful for functions that don't return anything. Only `undefined` and `null` are valid `void` values.

```ts
function log(msg): void {
  console.log(msg);
}
```

The `any` type is a catch all for everything. It is useful when a type is variable or unknown, but should be used sparingly.

```ts
const something: any;
```

## Arrays

The values in an array should be typed. There are two syntaxes for describing this:

```ts
const one: Array<string> = ['one', 'two', 'three'];
const two: string[] = ['uno', 'dos', 'tres'];
```

The first uses TypeScript's generics while the second is a special syntax.

## Functions

The arguments passed to a function and a function's return value should have types.

```ts
function power(n: number, p: number): number{
  return Math.pow(n, p);
}
```

## Interface

An object's structure is described through an `interface`. The `interface` consists of property names and their respective types.

A question mark after the property name designates the proeprty as optional.

```ts
interface Book {
  title: string;
  author: string;
  pageCount?: number; // optional
}
```

Function properties can be written two ways:

```ts
interface API {
  one(): boolean;
  two: () => boolean;
}
```

### Extend

An interface can extend another interface. This means that the interface inherits the properties from the interface that it extends.

```ts
interface Parent {
  name: string;
  color: string;
}

interface Child extends Parent {
  shape: string;
}

const c: Child = {
  name: "test",
  color: "red",
  shape: "trapezoid
};
```

## Multiple Types

### Intersection

If a variable can be a combination of types (interfaces/classes), it is desribed as an intersection of those types.

```ts
interface A {
  one: string;
}

interface B {
  two: string;
}

const c: A & B {
  one: "one",
  two: "two"
};
```

### Unions

If a variable can be one of a number of types, it is described as a union of those types.

```ts
const x: number | string = "1"; // ;)
```

Unions are useful when using strict null checks to specify that a variable can also be either `undefined` or `null`.

```ts
let n : number | undefined;
```

## Type

The `type` specifier can be used to describe complex types or alias other types.

```ts
type Union = number | string;
const x: Union = 2;

type Intersection = A & B;
const y: Intersection = {
  one: "uno",
  two: "dos"
};
```
