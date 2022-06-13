# Type Inference

## Avoid cluttering your code with inferable types

Declaring your number variable with its type or describing the type of an object when declaring it doesn't make sense at
all, instead it just adds noise.

```typescript
// Bad
let x: number = 12

// Good
let x = 12


//Bad
const person: {
  name: string
  born: {
    where: string
    when: string
  }
} = {
  name: 'John Doe',
  born: {
    where: 'Nigeria',
    when: 1969
  }
}

// Good
const person = {
  name: 'John Doe',
  born: {
    where: 'Nigeria',
    when: 1969
  }
}
```

What's true for objects is also true for arrays. TS has no trouble figuring out the return type of this function, based
on its inputs and operations.

```typescript
function square(nums: number[]) {
  return nums.map(x => x * x)
}

const squares = square([1, 2, 3, 4]) // Type is number[]
```

TS may even infer something more precise than what you expected at times.

```typescript
const axis1: string = 'x' // Type is string
const axis2 = 'y' // Type is "y"
```

Allowing types to be inferred can also facilitate refactoring. Say you have a **Product** type and a function to log it.

```typescript
interface Product {
  id: number
  name: string
  price: number
}

function logProduct(product: Product) {
  const id: number = product.id
  const name: string = product.name
  const price: number = product.price
  console.log(id, name, price)
}
```

At some point you learn that product IDs might have letters in them as well, and you change the type of **id** in
**Product** but this produces an error now (`Type 'string' is not assignable to type 'number'`).

A better implementation of `logProduct` would use destructing assignment:

```typescript
function logProduct(product: Product) {
  const {id, name, price} = product
  console.log(id, name, price)
}
```

In some situations, where TS doesn't have enough context to determine the type on its own, explicit type annotations are
welcomed.

> Some languages will infer types for parameters based on their eventual usage, but TS does not. In TS, a variable's type
> is generally determined when it is first introduced.
>
> Ideal TS code includes type annotations for function/method signatures but not for the local variables created in their
> bodies.

Parameter types can usually be inferred when the function is used as a callback for a library with type declarations:

```typescript
app.get('/health', (request, response) => { // request is express.Request, response is express.Response
  response.send('OK')
})
```

There are a few situations, though, where you might still want to specify a type even if it can be inferred.
To enable excess property checking for an object literal for example.

Same goes for a function's return type. You may still want to annotate this even if it can be inferred to ensure that
implementation errors don't leak out into uses of the function.

When you annotate the return type, it keeps implementation errors from manifesting as errors in user code, helps with
presentation of the function's contract in general.


## Use different variables for different types

There's a key insight about variables in TS: __while a variable's value can change, its type generally does not__. The
one common way a type can change is to narrow, but this involves a type getting smaller, not expanding to include new
values.

```typescript
// Not so good
let id: string|number = "12-34-56"
fetchProduct(id)

id = 123456
fetchProductBySerialNumber(id)
```

When a union type does work, it may create more issues down the road. The better solution is to introduce a new variable

```typescript
const serial = 123456
fetchProductBySerialNumber(serial)
```

In general, try to avoid type-changing variables.


## Understanding type widening

At runtime every variable has a single value. But at static analysis time, a variable has a set of __possible values__ -
its type. When you declare a constant without a type, the type checker needs to decide on a set of possible values from
the single value specified – this process is known as **__widening__**.

Suppose, you have a function to return the value of a 3D vector:

```typescript
interface Vector3 {
  x: number
  y: number
  z: number
}
function getComponents(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis]
}
```

But when you try to use it, TS flags an error:

```typescript
let x = 'x'
let vec = {x: 10, y: 20, z: 30}
getComponent(vec, x) // Argument of type 'string' is not assignable to parameter of type 'x' | 'y' | 'z'
```

The issue is that `x`'s type is inferred as **string**, whereas the `getComponent` function expected a more specific
type for its second argument. This is widening at work.

This process is ambiguous in the sense that there are many possible types for any given value. In this statement, for
example:

```typescript
const mixed = ['x', 1]
```

there are at least 9 possible types that can annotate this constant. Without more context, TS has no way to know which
one is "right".

> TS gives you a few ways to control the process of widening.
> One is **const**. If you declare a variable with **const** instead of **let**, it gets a narrower type.

TS is trying to strike a balance between specificity and flexibility. It needs to infer a specific enough type to catch
errors, but not so specific that it creates false positives.

If you know better, there are a few ways to override TS's default behavior. One is to supply an explicit type.

Another is to provide additional context to the type checker (e.g. by passing the value as the parameter to a function).

Third way is with a **const** assertion. This is a purely type-level construct.

```typescript
const v1 = { x: 1, y: 2 } // Type is { x: number; y: number }
const v2 = { x: 1 as const, y: 2 } // Type is { x: 1; y: number }
```

When you write `as const` after a value, TS will infer the narrowest possible type for it. No widening.


## Understand type narrowing

The opposite of widening is narrowing. Perhaps the most common example of this is null checking. You can also narrow a
variable's type for the rest of a block by throwing or returning from a branch.

Using **instanceof** works as well. So does property checking. Some built-in function such as **Array**.`isArray` are
abel to narrow types.

Another common way to help the type checker narrow your types is by putting an explicit 'tag' on them:

```typescript
interface UploadEvent {
  type: 'upload'
  filename: string
  contents: string
}
interface DownloadEvent {
  type: 'upload'
  filename: string
}
type AppEvent = UploadEvent | DownloadEvent

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e // Type is DownloadEvent
      break
    case 'upload':
      e // Type is UploadEvent
      break
  }
}
```

This pattern is known as a "tagged union" or "discriminated union", and it is ubiquitous in TS.

You can also introduce a custom function to help TS figure out a type:

```typescript
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el
}

function getElementValue(el: HTMLElement) {
  if (isInputElement(el)) {
    el // type is HTMLInputElement
    return el.value
  }
  el // Type is HTMLElement
  return el.textContent
}
```
This is known as "user-defined type guard". The **el is HTMLInputElement** as a return type tells the type checker that
it can narrow the type of the parameter if the function returns true.


## Create Objects All at Once

...You can also use the object spread operator to build up objects by field in a type-safe way. The key is to use a new
variable on every update so that each gets a new type:

```typescript
const pt0 = {}
const pt1 = {...pt0, x: 3}
const pt: Point = {...pt1, y: 4} // OK
```

While this is a roundabout way to build up such a simple object, it can be a useful technique for adding properties to
an object and allowing TS to infer a new type.

To conditionally add a property in a type-safe way, you can use object spread with **null** or `{}`, which add no
properties:

```typescript
declare let hasMiddle: boolean
const firstLast = {first: 'Harry', last: 'Truman'}
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})}
```


## Be consistent in your use of aliases

When you introduce a new name for a value:

```typescript
const borough = {name: 'Brooklyn', location: [40.688, -73.979]}
const loc = borough.location
```

you have created an *alias*. Changes ot properties on the alias will be visible on the original value as well.

```typescript
> loc[0] = 0
> borough.location
[0, -73.979]
```

Aliases are the bane of compiler writers in all languages because they make control flow analysis difficult.

...

TS's control flow analysis tends to be quite good for local variables. But for properties you should be on guard:

```typescript
function fn(p: Polygon) { /* ... */ }

polygon.bbox // Type is BoundingBox | undefined
if (polygon.bbox) {
  polygon.bbox // Type is BoundingBox
  fn(polygon)
  polygon.bbox // Type is still BoundingBox
}
```

The call to `fn(polygon)` could very well un-set **polygon.bbox**, so it would be safer for the type to revert to
**BoundingBox | undefined**. But this would get frustrating: you'd have to repeat your property checks every time you
called a function. So TS makes the pragmatic choice to assume the function does not invalidate its type refinements.
But it *could*. If you'd factored out a local **bbox** variable instead of using **polygon.bbox**, the type of **bbox**
would remain accurate, but it might no longer be the same value as **polygon.bbox**.


## Use async functions instead of callbacks for asynchronous code

When you target ES5 or earlier, the TS compiler will perform some elaborate transformations to make **async** and
**await** work. In other words, whatever your runtime, with TS you can use **async/await**.

Type inference also works well with **Promise.race**, which resolves when the first of its input Promises resolves. You
can use this to add timeouts to Promises in a general way.

<details>
  <summary>Adding timeouts to Promises example</summary>

  ```typescript
  function timeout(ms: number): Promise<never> {
    return new Promise((res, rej) => {
      setTimeout(() => reject('timeout'), ms)
    })
  }

  async function fetchWithTimeout(url: string, ms: number) {
    return Promise.race([fetch(url), timeout(ms)])
  }
  ```

  The return type of `fetchWithTimeout` is inferred as **Promise<Response>**, no type annotations required. It's
  interesting to dig into why this works: the return type of **Promise.race** is the union of the types of its inputs, in
  this case **Promise<Response | never>**. But taking a union with a **never** is a no-op, so this gets simplified to
  **Promise<Response>**. 

</details>


> A function should either always be run synchronously or always be run asynchronously. It should never mix the two.


## Use functional constructs and libraries to help types flow

Use built-in functional constructs and those in utility libraries like Lodash instead of hand-rolled constructs to
improve type flow, increase legibility, and reduce the need for explicit type annotations.
