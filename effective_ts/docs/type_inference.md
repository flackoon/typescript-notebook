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
