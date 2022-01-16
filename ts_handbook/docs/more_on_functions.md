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


## Optional Parameters

A parameter can be made _optional_ with `?:`

```typescript
function f(x?: number) {
  // ...
}

f();   // OK
f(10); // OK
```

Although the parameter is specified as type **number**, the `x` parameter will actually have the type **number | undefined**
because unspecified parameters in JS get the value **undefined**.


### Optional Parameters in Callbacks

In JS, if you call a function with more arguments than there are parameters, the extra arguments are simply **ignored**.
TS behaves the same way. Functions with fewer parameters (of the same types) can always take the place of functions with
more parameters.

> When writing a function type for a callback, _never_ write an optional parameter unless you intend to call the function 
> **without** passing that argument.


## Function Overloads

In TS, we can specify a function that can be called in different ways by writing _overload signatures_. To do this, 
write some number of function signatures, followed by the body of the function:

```typescript
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y != undefined) {
    return new Date(y, mOrTimestamp, d);
  } 
  
  return new Date(mOrTimestamp);
}

const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);

// No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

The first two signatures are the _overload signatures_. The function implementation follows with a compatible signature.
Functions have an _implementation_ signature, but this signature can't be called directly. Even though we wrote a 
function with two optional parameters after the required one, it can't be called with two parameters!


### Overload Signatures and the Implementation Signature

> The signature of the _implementation_ is **not** visible from the _outside_. When writing an overloaded function, you
> should always have _two_ or more signatures above the implementation of the function.

The implementation signature must also be _compatible_ with the overload signatures.


### Writing Good Overloads

There are a few guidelines you should follow when using function overloads. Following these principles will make your
function easier to call, easier to understand, and easier to implement.

Let's consider a function that returns the length of a string or an array:

```typescript
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

The function is fine; it can be invoked with strings or arrays. However, it can't be invoked with a value that might be
a string _or_ an array, because TS can only resolve a function call to a single overload:

```typescript
len("")     // OK
len([0])    // OK
len(Math.random() > 0.5 ? "hello" : [0]);   // Not OK
```

Because both overloads have the same argument count and the same return type, we can instead write a non-overloaded
version of the function:

```typescript
function len(x: any[] | string) {
  return x.length;
}
```

Callers can invoke this with either sort of value, and as an added bonus, we don't have to figure out a correct implementation
signature.

> Always prefer parameters with union types instead of overloads when possible.


### Declaring **this** in a Function

TS will infer what the **this** should be in a function via code flow analysis, for example:

```typescript
const user = {
  id: 123,
  admin: false,
  becomeAdmin: () => {
    this.admin = true;
  }
}
```

TS understands that the function `user.becomeAdmin` has a corresponding **this** which is the outer object `user`. This 
can be enough for a lot of cases, but there are a lot of cases where you need more control over what object **this**
represents. The JS specification states that you cannot have a parameter called **this**, and so TS uses that syntax
space to let you declare the type for **this** in the function body.

```typescript
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

This pattern is common with callback-style APIs, where another object typically controls when your function is called.
Note that you need to use `function` and not arrow functions to get this behavior.


## Other Types to Know About


### void

**void** represents the return value of functions which don't return a value. It's the inferred type any time a function
doesn't have any **return** statements, or doesn't return any explicit value from those return statements (`return;`).

> In JS, a function that doesn't return any value will return **undefined** implicitly. However, **void** and **undefined**
> are not the same thing in TS. 


### object

The special type **object** refers to any value that isn't a primitive (string, number, bigint, boolean, symbol, null,
or undefined). This is different from the _empty object type { }_, and also different from the global type **Object**. 
It's very likely you will never use Object.

> In JS, function values are objects: They have props, prototype chain, are instance of Object and so on. For the same 
> reason function types are considered to be **objects** in TS.


### unknown

The **unknown** type represents **any** value. This is similar to the **any** type, but is safer because it's not legal
to do anything with an **unknown** value:

```typescript
function f1(a: any) {
  a.b();  // OK
}

function f2(a: unknown) {
  a.b();  // NOT OK
}
```


### never

Some functions **never** return a value:

```typescript
function fail(msg: string): never {
  throw new Error(msg);
}
```

The **never** type represents values which are **never** observed. In a return type, this means that the function throws 
an exception or terminates execution of the program.

**never** also appears when TS determines there's nothing left in an union.


### Function 

The global type **Function** describes properties like `bind`, `call`, `apply`, and others present on all function values
in JS. It also has the special property that values of type `Function` can always be called; these calls return **any**:

> If you need to accept an arbitrary function but don't intend to call it, the type `() => void` is generally safer.


## Rest Parameters and Arguments


### Rest Parameters

A rest parameter appears after all others and uses the `...` syntax:

```typescript
function multiply(n: number, ...m: number[]) {
  return m.map(x => n * x);
}

// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

In TS, the type annotation on these parameters is implicitly **any[]** instead of **any**, and any type annotation given
must be of the form **Array<T>** or **T[]**, or a tuple type.


### Rest Arguments

Conversely, we can provide a variable number of arguments from an array using the spread syntax. For example, the **push**
method of arrays takes any number of arguments:

```typescript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```


## Parameter Destructing

The type annotation for the object goes after the destructing syntax:

```typescript
function sum({ a, b, c}: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```


## Assignability of Functions


### Return type void

The **void** return type for functions can produce some unusual, but expected behavior.

Contextual typing with a return type of **void** does **not** enforce functions to **not** return something. Another way
of saying this is a contextual function type with a **void** return type (type vf = () => void), when implemented, can 
return **any** other value, but it will be ignored.

When the return value of one of these functions is assigned to another variable, it will retain the type of **void**: