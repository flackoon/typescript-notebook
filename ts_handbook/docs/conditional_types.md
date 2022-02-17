# Conditional Types

- [Conditional Types](#conditional-types-1)
  - [Conditional Type Constraints](#conditional-type-constraints)
  - [Inferring Within Conditional Types](#inferring-within-conditional-types)
- [Distributive Conditional Types](#distributive-conditional-types)

## Conditional Types

Conditional types help describe the relation between the types of inputs and outputs.

```typescript
interface Animal {
  live(): void;
}

interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string; // number
```

Conditional types take a form that looks a little like conditional expressions in JS:
`SomeType extends OtherType ? TrueType : FalseType;`

The power of conditional types comes from using them with generics.

If you have an overloaded function, for example:

```typescript
interface IdLabel {
  id: number /* some fields */;
}

interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

But there are 2 things to notice:

1. If a library has to make the same sort of choice over and over throughout its API, this becomes cumbersome.
2. We have to create 3 overloads handling each case.

Instead, we can encode that logic in a conditional type:

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

We can then use that conditional type to simplify the overloads down to a single function with no overloads.

```typescript
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  // ...
}
```

### Conditional Type Constraints

Just like with narrowing with type guards can give us a more specific type, the true branch of a conditional type will
further constrain generics by the type we check against.

For example, let's take the following:

```typescript
type MessageOf<T> = T["message"];
// “Type '"message"' cannot be used to index type 'T'.”
```

TS errors simply because **T** isn't known to have a property called **message**. We could constrain **T**, and TS
would no longer complain:

```typescript
type MessageOf<T extends { message: unknown }> = T["message"];
```

We could also write a type called **Flatten** that flattens array types to their element types, but leaves alone otherwise:

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;

type Str = Flatten<string[]>; // type is String
type Num = Flatten<number>;   // type is Number
```

### Inferring Within Conditional Types

Conditional types provide us with a way to infer types we compare against in the true branch using the **infer** keyword.
For example, we could have inferred the element type in **Flatten** instead of fetching it out "manually" with an
indexed access type:

```typescript
type Flatten<Type> = Type extends <Array infer Item> ? Item : Type;
```

Here, we use the **infer** keyword to declaratively introduce a new generic type variable named **Item** instead of 
specifying how to retrieve the element type of **T** within the true branch. This frees us from having to think about
how to dig through and probing apart the structure of the types we're interested in.

Some useful helper type aliases can be written using the **infer** keyword.
For example,  for simple cases, we can extract the return type out of function types:

```typescript
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return ? Return : never;

type Num = GetReturnType<() => number>; // type is number
```

When inferring from a type with multiple call signatures (such as the type of an overloaded function), inferences are 
made from the *last* signature (which, presumably, is the most permissive catch-all case).

> It is not possible to perform overload resolution based on a list of argument types.

```typescript
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>; // type is string | number
```

## Distributive Conditional Types

When conditional types act on a generic type, they become __distributive__ when given a union type. 

```typescript
type ToArray<Type> = Type extends any ? Type[] : never;
```

If we plug a union type into **ToArray**, then the conditional type will be applied to each member of that union.

```typescript
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>;
// type StrArrOrNumArr = string[] | number[]
```

Typically, distributivity is the desired behavior. To avoid that behavior, you can surround each side of the **extends**
keyword with square brackets.

```typescript
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

type StrArrOrNumArr = ToArrayNonDist<string | number>;
// type StrArrOrNumArr = (string | number)[]
