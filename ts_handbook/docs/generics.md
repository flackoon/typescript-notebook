# Generics

- [Hello World of Generics](#hello-world-of-generics)
- [Generic Types](#generic-types)
- [Generic Classes](#generic-classes)
- [Generic Constrains](#generic-constrains)
- [Using Type Parameters in Generic Constraints](#using-type-parameters-in-generic-constraints)
- [Using Class Types in Generics](#using-class-types-in-generics)

## Hello World of Generics

The identity function is a function that will return back whatever is passed in.
Without generics, we would either have to give the identity function a specific type:

```typescript
function identity(arg: number): number {
  return arg;
}
```

Or, we could describe the identity function using the **any** type:

```typescript
function identity(arg: any): any {
  return arg;
}
```

Instead, we need a way of capturing the type of the argument in such a way that we can also use it to denote what is
being returned. Here, we will use a _type variable_, a special kind of variable that works on types rather than values.

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}
```

Once written like this, we can call the generic identity function in one of two ways. The first way is to pass all of
the arguments, including the type argument, to the function:

```typescript
let output = identity<string>("myString");
// let output: string
```

The second way is also perhaps the most common. Here we use _type argument inference_ – that is, we want the compiler to
set the value of **Type** for us automatically based on the type of the argument we pass in.

```typescript
let output = identity("myString");
// let output: string
```

> [**NOTE**]: While type argument inference can be a helpful tool to keep code shorter and more readable, you may need to
> explicitly pass in the type argument as we did in the previous example when the compiler fails to infer the type, as
> may happen in more complex examples.

## Generic Types

We may want to move the generic parameter to be a parameter of the whole interface. This lets us see what type(s) we're
generic over. This makes the type parameter visible to all other members of the interface.

```typescript
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

We now have a non-generic function signature that is a part of a generic type. When we use **GenericIdentityFn**, we now
will also need to specify the corresponding type argument (here: **number**), effectively locking in what the underlying
call signature will use.

In addition to generic interfaces, we can also create generic classes. Note that it is not possible to create generic
enums and namespaces.

## Generic Classes

A generic class has a similar shape to a generic interface. Generic classes have a generic type parameter list in angle
brackets (`<>`) following the name of the class.

```typescript
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

This is a pretty literal use of the **GenericNumber** class, but nothing is restricting it to only use the **number**
type. We could have used **string** or even more complex objects.

Just as with interface, putting the type parameter on the class itself lets us make sure all of the properties of the
class are working with the same type.

## Generic Constrains

```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // now we know it has a .length property
  return arg;
}
```

Because the generic function is now constrained, it will no longer work over any and all types:

```typescript
loggingIdentity(3);
// “Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.”
```

Instead, we need to pass in values whose type has all the required properties:

`loggingIdentity({length: 10, value: 3});`

## Using Type Parameters in Generic Constraints

You can declare a type parameter that is constrained by another type parameter. For example, here we'd like to get a
property from an object given its name. We'd like to ensure that we're not accidentally grabbing a property that does not
exist on the **obj**, so we'll place a constraint between the 2 types:

```typescript
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m"); // Argument of type '"m"' is not assignable to parameter of type '"a"' | "b" | "c" | "d"'.
```

## Using Class Types in Generics

When creating factories in TS using generics, it is necessary to refer to class types by their constructor functions. FOr
example,

```typescript
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```
