# TypeScript's Type system

## Prefer Type Declarations to Type Assertions

Type declarations are to be preferred over type assertions because they verify that the value conforms to the type
specified. If it doesn't, TS flags an error, while type assertion would silence the error by telling the type checker
that, for whatever reason, you know better than it does.

In case you are ever wondering how to use declaration with arrow functions:

```typescript
const people: Person[] = ['alice', 'bob', 'jan'].map((name): Person => ({name})) // type is Person[]
```

This performs all the same checks on the value. The parentheses are significant here! **(name): Person** infers the type
of **name** and specifies that the returned type should be **Person**. 

On the topic of type assertions: You should only use these in a context that isn't available to the type checker (like a
DOM tree) and you know for sure more precisely the type of the value you are about to assert.

This also applies to non-null assertions. Only do these assertions when you are 101% sure that you know TS doesn't know
something you do.


## Recognize the Limits of Excess Property Checking

When assigning an object literal to a variable with a declared type, TS makes sure it has the properties of that type
and __no others__:

```typescript
interface Room {
  numDoors: number
  ceilingHeightFt: number
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // 'elephant' doesn't exist on type 'Room'
}
```

In that scenario, TS will flag an error. If you introduce an intermediate variable though, things get different:

```typescript
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // 'elephant' doesn't exist on type 'Room'
}

const r: Room = obj // No errors
```

The type of **obj** is inferred as `{numDoors: number; ceilingHeightFt: number; elephant: string}` because this type
includes a subset of the values in the **Room** type, it is assignable to **Room**, and the code passes the type checker.

So where's the difference?

> In the first example, we've triggered a process known as **excess property checking**, which helps catch an important
> class of errors that the structural type system would otherwise miss.

Excess property checking is an effective way of catching typos and other mistakes in property names that would otherwise
be allowed by the structural typing system. It's particularly useful with types like **Options** that contain optional
fields. But it also limited in scope: only applies to object literals.


## Apply Types to Entire Function Expressions When Possible

Function types' first advantage is reducing repetition.

```ts
type BinaryFn = (a: number, b: number) => number
const add: BinaryFn = (a, b) => a + b
const sub: BinaryFn = (a, b) => a - b
```

Makes the logic more apparent, plus we've gained a check that the return type of all the function expressions is **number**.

Libraries often provide types for common function arguments. For example, ReactJS provides a **MouseEventHandler** type
that can be applied to an entire function rather than specifying **MouseEvent** as a type for the function's parameter.

Applying a type to a function expression is also helpful when matching a signature of some other function. For example
the browser's native **fetch** function. It's return type is `Promise<Response>`, and to get the response from a fetch
request, you need to do `.json()` or `.text()` which might result in an error if the request was actually unsuccessful.

So we can implement our own **checkedFetch** function to do the status check for use. The type declarations for **fetch**
in **lib.dom.d.ts** look like so:

```typescript
declare function fetch(
  input: RequestInfo, init?: RequestInit
): Promise<Response>
```

so we can write our own function like this:

```typescript
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init)
  if (!response.ok) throw new Error('Error!')
  return response
}
```

or even more concisely:

```typescript
const checkedFetch: typeof fetch = async (input) => {
  const response = await fetch(input, init)
  if (!response.ok) throw new Error('Error!')
  return response
}
```

We've changed from a function statement to a function expression and applied a type (**typeof fetch**) to the entire
function. This allows TS to infer the types of the **input** and **init** parameters.


## Know the Difference between type and interface

An interface can extend a type, and a type can extend an interface. Except for complex types (like a union type) - an
interface cannot extend these.

A class can implement either an interface or a simple type.

```ts
type NamedVariable = (Input | Output) & { name: string }
```

This ^ cannot be expressed with an interface. A type is, in general, more capable than an interface. It can be a union,
and it can also take advantage of features like mapped or conditional types.

For example, you can express something like a tuple using an interface, but it's awkward and drops all the tuple methods
like `concat`.

An interface has some things that a type doesn't, however. One of them is that it can be __augmented__.

```ts
interface IState {
  name: string
  capital: string
}

interface IState {
  population: number
}

const wyoming: IState = {
  name: 'A',
  capital: 'B',
  population: 500_000,
} // OK
```

This is known as **declaration merging**.

> If it's essential that no one ever augmented your type, then use **type**.

