# Narrowing

- [typeof type guards](#typeof-type-guards)
- [Truthiness narrowing](#truthiness-narrowing)
- [Equality narrowing](#equality-narrowing)
- [The `in` operator narrowing](#the-in-operator-narrowing)
- [Assignments](#assignments)
- [Control flow analysis](#control-flow-analysis)
- [Using type predicates](#using-type-predicates)
- [Discriminated unions](#discriminated-unions)
- [The never type](#the-never-type)
- [Exhaustiveness checking](#exhaustiveness-checking)

Within our `if` checks for a variable type, TS sees `typeof something === "number"` and understands that as a special form
of code called _type guard_. TS follows possible paths of execution that our programs can take to analyze the most specific
possible type of a value at a given position. It looks at these special checks (_type guards_) and assignments, and the
process of refining types to more specific types than declared is called _narrowing_.

There are a couple of different constructs TS understands for narrowing.

## typeof type guards

TS expects this to return a certain set of strings:

- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

Because TS encodes how **typeof** operates on different values, it knows about some of its quirks in JS.

```typescript
const printAll = (strs: string | string[] | null) => {
  if (typeof strs === "object") {
    for (const s of strs) {
      // Error: Object is possibly 'null'
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
};
```

TS lets us know that `strs` was only narrowed down to **string[] | null** instead of just **string[]**.

## Truthiness narrowing

In JS, constructs like _if_ first "coerce" their conditions to **booleans** to make sense of them, and then choose their
branches depending on whether the result is **true** or **false**. Values like

- 0
- NaN
- ""
- 0n
- null
- undefined

all coerce to **false**, and other values get coerced to **true**.

It's fairly popular to leverage this behavior, especially for guarding against values like **null** or **undefined**.

```typescript
const printAll = (strs: string | string[] | null) => {
  if (strs && typeof strs === "object") {
    // for ...
  } else if (typeof strs === "string") {
    // ...
  }
};
```

Keep in mind though that truthiness checking on primitives can often be error prone.

```typescript
function printAll(strs: string | string[] | null) {
  // DON'T DO THIS
  if (strs) {
    // type guards ...
  }
}
```

We wrapped the entire body of the function in a truthy check, but this has a subtle downside: we may no longer be
handling the empty string case correctly.

## Equality narrowing

TS also uses **switch** statements and equality checks like `===`, `!==`, `==`, and `!=` to narrow types.

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
  }
}
```

When we checked that both **x** and **y** are both equal in the above example, TS knew their types also had to be equal.
Since **string** is the only common type, TS knows that **x** and **y** must be a **string** in the first branch.

Checking against specific literal values (as opposed to variables) works also.

```typescript
function printAll(strs: string | string[] | null) {
  if (strs !== nul) {
    // ...
  }
}
```

JS's looser equality checks with `==` and `!=` also get narrowed correctly. Checking whether something == **null** actually
not only checks whether it is specifically the value **null** – it also checks whether it's potentially **undefined**.
The same applies to == **undefined**: it checks whether a value is either **null** or **undefined**.

```typescript
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    // ... handle not null/undefined value
  }
}
```

## The `in` operator narrowing

TS takes the JS `in` operator into account as a way to narrow down potential types.

For example, with the code `"value" in x` where "value" is a string literal and `x` is a union type. The "true" branch
narrows x's types which have either an optional or required property **value**, and the "false" branch narrows to types
which have an optional or missing property **value**.

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

To reiterate optional properties will exist in both sides of narrowing, for example a human could both swim and fly (with
the right equipment) and thus should show up in both sides of the `in` check:

```typescript
// continuing prev example
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    // animal: Fish | Human
  } else {
    // animal: Bird | Human
  }
}
```

## Assignments

Assignability is always checked against the declared type.

## Control flow analysis

The analysis of code based on reachability is called **control flow analysis**, and TS uses this flow analysis to narrow
types as it encounters type guards and assignments. When a variable is analyzed, control flow can split off and re-merge
over and over again, and that variable can be observed to have a different type at each point.

## Using type predicates

To define a user-defined type guard, we simply need to define a function whose return type is a _type predicate_:

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

`pet is Fish` is our type predicate in this example. A predicate takes the form `parameterName is Type`, where **parameterName**
must be the name of a parameter from the current function signature.

## Discriminated unions

When every type in a union contains a common property with literal types, TS considers that to be a _discriminated union_,
and can narrow out the members of the union. In such case a common property is called _discriminant_.

## The never type

When narrowing, you can reduce the options of a union to a point where you have removed all posibilities and have nothing
left. In those cases, TS will use a **never** type to represent a state which shouldn't exist.

## Exhaustiveness checking

The **never** type is assignable to every type; however, no type is assignable to **never**. This means you can use
narrowing and rely on **never** turning up to do exhaustive checking in a switch statement.

For example, adding a default to a function which tries to assign a parameter to **never** will raise when every possible
case has not been handled.

```typescript
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

Adding a new member to the **Shape** union, will cause a TS error.
