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