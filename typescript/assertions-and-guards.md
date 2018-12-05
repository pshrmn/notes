# Type Assertions and Guards

## Type Assertions

TypeScript can be forced to treat a variable as a certain type. There are two syntaxes for this.

```ts
const thing = {
  title: "The Cat in the Hat",
  author: "Dr. Seuss"
};
(<Book>thing).title
// or
(thing as Book).author
```

## Type Guards

Unions are convenient, but sometimes it is useful to narrow down the type.

```ts
interface One {
  shape: string;
  size: number
}

interface Two {
  shape: string;
  color: string;
}

type OneOrTwo = One | Two;

const x: OneOrTwo = { shape: "square", size: 2 };
```

In the above code, we can see that `x` is a `One` because it has a `size` property. If we attempt to access `x.size`, TypeScript won't know for sure that `x` has a `size` property because it might be a `Two`. Type assertions can be used to safely access a property.

```ts
const size: number = (x as One).size;
```

Type guards are functions that assert that a variable is a certain type. When the below function returns `true`, TypeScript will treat the variable in the scope where the function as called as a `One` (and a `Two` if it returns `false`).

```ts
function isOne(v: OneOrTwo): v is One {
  return (v as One).size !== undefined;
}

const x: OneOrTwo = { shape: "square", size: 2 };
if (isOne(x)) {
  // no assertion necessary
  console.log(x.size);
} else {
  console.log(x.color);
}
```
