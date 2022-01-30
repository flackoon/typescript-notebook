# Indexed Access Types

We can use an *indexed type* to look up specific property on another type:

```typescript 
type Person = { age: number; name: string; alive: boolean };\
type Age = Person["age"]; // number
```

The indexing type is itself a type, so we can use unions, keyof, or other types entirely:


```typescript
type I1 = Person["age" | "name"]; // string | number
type I2 = Person[keyof Person];   // string | number | boolean
```

Another example of indexing with an arbitrary type is using **number** to get the type of an array's elements. We can
combine this with **typeof** to conveniently capture the element type of an array literal:

```typescript
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];

type Person = typeof MyArray[number];
// type Person = {
//     name: string;
//     age: number;
// }

type Age = typeof MyArray[number]["age"];
// type Age = number
// Or
type Age2 = Person["age"];
// type Age2 = number
```

You can only use types when indexing, meaning you can't use a **const** to make a variable reference:

```typescript
const key = "age";s
type Age = Person[key];
// type 'key' cannot be used as an index type.
// 'key' referes to a value, but is being used as a type here. Did you mean 'typeof key'?
```

However, you can use an alias for a similar style of refactor:

```typescript
type key = "age";
type Age = Person[key];
```
