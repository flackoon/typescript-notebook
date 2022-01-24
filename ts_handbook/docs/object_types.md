# Object Types

## Property Modifiers

Each prop in an object type can specify a couple of things: the type, whether the prop is optional, and whether the prop
can be written to.

### Optional Properties

We mark a property as _optional_ by adding a question mark (`?`) to the end of its name.

```typescript
interface PaintOptions {
  shape: Shape;
  xPos?: number;
}
```

We can read from those properties – but when we do under `strictNullChecks`, TS will tell us they're potentially **undefined**.

### **readonly** Properties

Props can also be marked as **readonly** for TS. While it won't change any behavior at runtime, a property marked as
**readonly** can't be written to during type-checking.

```typescript
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  obj.prop = "hello"; // Cannot assign to 'prop' because it is a read-only property.
}
```

Using the **readonly** modifier doesn't necessarily imply that a value is totally immutable – or in other words, that its
internal contents can't be changed. It just means the prop itself can't be re-written to.

It's important to manage expectations of what readonly implies. It's useful to signal intent during development time for
TS on how an object should be used. TS doesn't factor in whether properties on two types are **readonly** when checking
whether those types are compatible, so **readonly** props can also change via aliasing.


### Index Signatures

Sometimes you don't know all the names of a type's properties ahead of time, but you do know the shape of the values.

In those cases you can use index signature to describe the types of possible values, for example:

```typescript
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
```

An index signature property must be either 'string' or 'number'.

While string index signatures are a powerful way to describe the "dictionary" pattern, they also enforce that all 
properties match their return type. This is because a string index declares that `obj.property` is also available as
`obj["property"]`. In the following example, `name`'s type does not match the string index's type, and the type checker
gives an error:

```typescript
interface NumberDictionary {
  [index: string]: number;
  length: number; // OK
  name: string; // Property 'name' of type 'string' is not assignable to 'string' index type 'number'.
}
```

You can also make index signatures **readonly** in order to prevent assignment to their indices.

```typescript
interface ReadONlyStringArray {
  readonly [index: number]: string;
}
```


## Extending Types

The **extends** keyword on an **interface** allows us to effectively copy members from other named types, and add 
whatever new members we want. 

```typescript
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

Interfaces can also extend from multiple types.

```typescript
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}
```


## Intersection Types

An intersection type is defined using the `&` operator.

```typescript
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```

This will produce a new type that has all the members of **Colorful** and **Circle**.


## Generic Object Types

```typescript
interface Box<Type> {
  contents: Type;
}

let box: Box<string>;
```

**Box** is reusable in that **Type** can be substituted with anything. That means that when we need
a box for a new type, we don't need to declare a new **Box** type at all. 

This also means that we can avoid overloads entirely by instead using generic functions.

```typescript
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

Type aliases can also be generic. 


```typescript
interface Box<Type> {
  contents: Type;
}

// by using a type alias instead:
type Box<Type> = {
  contents: Type;
}
```


### The **Array** Type

Generic object types are often some sort of container type that work independently of the type of elements they contain.
It's ideal for data structures to work this way so that they're re-usable across different data types.

```typescript
function doSomething(value: Array<string>) {
  // ...
}
```

Modern JS also provides other data structures which are generic, like **Map<K, V>**, **Set<T>**, **Promise<T>**. All this 
really means is that because of how **Map**, **Set**, and **Promise** behave, they can work with any sets of types.


### The **ReadonlyArray** Type

This is a special type that describes arrays that shouldn't be changed.

Much like the **readonly** modifier for properties, it's mainly a tool we can use for intent. When we see a function that
returns **ReadonlyArray**s, it tells us we're not meant to change the contents at all, and when we see a function that 
consumes **ReadonlyArray**s, it tells us that we can pass any array into that function without worrying that it will change
its contents.

```typescript
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

Just as TS provides a shorthand syntax for **Array<Type>** with **Type[]**, it also provides a shorthand syntax for 
**ReadonlyArray<Type>** with **readonly Type[]**.

> [**NOTE**] Unlike the **readonly** property modifier, assignability isn't bidirectional between regular **Arrays**
> and **ReadonlyArrays**.
> 
> ```typescript
> let x: readonly string[] = [];
> let y: string[] = [];
> x = y;
> y = x; // Not allowed.
> ```


## Tuple Types

A _tuple type_ is another sort of **Array** type that knows exactly how many elements it contains, and exactly
which types it contains at specific positions.

`type StringNumberPair = [string, number];`

Tuple types are useful in heavily convention-based APIs, where each element's meaning is "obvious". This gives us flexibility
in whatever we want to name our variables when we destructure them. However, since not every user holds te same view of
what's obvious, it may be worth considering whether using objects with descriptive property names may be better for your API.

Simple tuple types are equivalent to types which are versions of **Array**s that declare properties for specific indexes,
and that declare **length** with a numeric literal type.

```typescript
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```

Tuples can have optional properties by writing out a question mark (`?` after an element's type). Optional tuple elements 
can only come at the end, and also affect the type of **length**.

Tuples cna also have rest elements, which have to be an array/tuple type.

```typescript
type StringNumberBooleans = [string, number, ...boolean[]]; // ["blah", 10, false, false, true]
type StringBooleansNumber = [string, ...boolean[], number]; // ["blah", false, true, false, 10]
type BooleansStringNumber = [...boolean[], string, number]; // [true, true, true, "false", 1]
```

Tuples types can be used in rest parameters and arguments, so the following:

```typescript
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

is basically equivalent to: 

```typescript
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```


### **readonly** Tuple Types

Tuples types have **readonly** variants, and can be specified by sticking a **readonly** modifier in front of them - just
like with array shorthand syntax.

Tuples tend to be created and left un-modified in most code, so annotating types as **readonly** tuples when possible is
a good default. This is also important given that array literals with const assertions will be inferred with readonly 
tuple types.