# More on Functions


## Function Type Expressions

The simplest way to describe a function is with a _function type expression_. These types are syntactically similar to
arrow functions:

```typescript
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```

> Note that the parameter name is **required**. The function type `(string) => void` means "a function with a parameter
> named string of type any"!


## Call Signatures

In JS, functions can have properties in addition to being callable. However, the function type expression syntax doesn't
allow for declaring properties. If we want to describe something callable with properties, we can write a _call signature_
in an object type:

```typescript
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
}

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

Note that the syntax is slightly different compared to a function type expression – use `:` between the parameter list
and the return type rather than `=>`.


## Construct Signatures

JS functions can also be invoked with the `new` operator. TS refers to these as _constructors_ because they usually create 
a new object. You can write a _construct signature_ by adding the `new` keyword in front of a call signature:

```typescript
type SomeConstructor = {
  new (s: string): SomeObject
}

function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

Some objects, like JS's _Date_ object, can be called with or without `new`. You can combine call and construct signatures
i the same type arbitrarily.

```typescript
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```


## Generic Functions

It's common to write a function where the types of the input relate to the type of the output, or where the types of
two inputs are related in some way. Let's consider for a moment a function that returns the first element of an array:

```typescript
function firstElement(arr: any[]) {
  return arr[0];
}
```

This function does its job, but unfortunately has the return type `any`. It'd be better if the function returned the
type of the array element.

In TS, _generics_ are used when we want to describe a correspondence between two values. We do this by declaring a _type
parameter_ in the function signature.

```typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arra[0];
}
```

By adding a type parameter `Type` to this function and using it in two places, we've created a link between the input of
the function and the output. Now when we call it, a more specific type comes out.


### Inference

Note that we didn't have to specify `Type` in this sample. The type was _inferred_ by TS.

We can use multiple type parameters as well. For example, a standalone version of `map` would look like this:

```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
```


### Constraints

We've written some generic functions that can work with _any_ kind of values. Sometimes we want to relate two values, but
can only operate on a certain subset of values. In this case, we can use a _constraint_ to limit the kinds of types a 
type parameter can accept.

Let's write a function that returns the longer of two values. To do this, we need a `length` property that's a number. We
_constrain_ the type parameter by writing an _extends_ clause:

```typescript
function longest<Type extends { length: number}>(a: Type, b: Type) {
  return a.length >= b.length ? a : b;
}
```

Because we constrained `Type` to `{ length: number }`, we were allowed to access the `.length` property of the `a` and `b`
parameters. Without the type constraint, we wouldn't be able to access those properties because the values might have been
some other type without a length property.


### Working with Constrained Values

Here's a common error when working with constraint types:

```typescript
function minimumLength<Type extends { length: number }>(obj: Type, minimum: numeber): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minumum };
  }
}
```

The problem is that the function promises to return the _same_ kind of object as was passed in, not just _some_ object
matching the constraint. 


### Specifying Type Arguments

TS can usually infer the intended type arguments in a generic call, but not always. For example, let's say you wrote a
function to combine two arrays:

```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

Normally it would be an error to call this function with mismatched arrays:

```typescript
const arr = combine([1, 2, 3], ["hello"]);
```

If you intended to do this, however, you could manually specify `Type`:

```typescript
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```


### Guidelines for Writing Good Generic Functions

Having too many type parameters or using constraints where they aren't needed can make inference less successful, frustrating
callers of your function.


#### Push Type Parameters Down

Here are two ways of writing a function that appear similar:

```typescript
function firstElement<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}

// a: number (good)
const a = firstElement([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]); 
```

> **Rule**: When possible, use the type parameter itself rather than constraining it


#### Use Fewer Type Parameters

Here's another pair of similar functions:

```typescript
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(Arr: Type, func: Func): Type[] {
  return arr.filter(func);
}
```

We've created a type parameter `Func` that _doesn't relate two values_. That's always a red flag, because it means callers
wanting to specify type arguments have to manually specify an extra type argument for no reason. `Func` doesn't do anything
but make the function harder to read and reason about.

> **Rule**: Always use as few type parameters as possible


#### Type Parameters Should Appear Twice

Sometimes we forget that a function might not need to be generic:

```typescript
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

// We could just as easily have written a simpler version
function greet2(s: string) {
  console.log(`Hello, ${s}`);
}
```

**Remember**, type parameters are for _relating the types of multiple values_. If a type parameter is only used once
in the function signature, it's not relating anything.

> **Rule**: If a type parameter only appears in one location, strongly reconsider if you actually need it


### Optional Parameters