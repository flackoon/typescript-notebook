# Getting to Know TypeScript

- [Getting to Know TypeScript](#getting-to-know-typescript)
  - [Know Which TypeScript Options You're Using](#know-which-typescript-options-youre-using)

## Know Which TypeScript Options You're Using

TS is most helpful when it has type information, so you should be sure to set **noImplicitAny** whenever possible. One 
you grow accustomed to all variables having types, TS without **noImplicitAny** feels almost like a different language.

**strictNullChecks** controls whether **null** and **undefined** are permissible values in every type.

## You Cannot Check TS Types at Runtime

You may be tempted to write code like this:

```typescript
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width ** 2;
  }
}
```

The **instanceof** check happens at runtime, but **Rectangle** is a type and so it cannot affect the runtime behavior of
the code. 
To ascertain the type of shape you're dealing with, you'll need some way to reconstruct its type at runtime. In this 
case you can check for the presence of a **height** property.

```typescript
function calculateArea(shape: Shape) {
  if ('height' in shape) { 
    return shape.width * shape.height  // Type is Rectangle
  } else {
    return shape.width ** 2 // Type is Square
  }
}
```

Another way would've been to introduce a "tag" to explicitly store the type in a way that's available at runtime:

```typescript
interface Square {
  kind: 'square';
  width: number;
}

interface Rectangle {
  kind: 'rectangle';
  height: number;
}

type Shape = Square | Rectangle

function calculateArea(shape: Shape) {
  if (shape.kind === 'rectangle') {
    // ... code
  }
}
```

> This is an example of a "tagged" union. Because they make it so easy to recover type information at runtime, tagged 
> unions are ubiquitous in TS.

Making **Square** and **Rectangle** classes would've been another way to fix the error:

```typescript
class Square {
  constructor(public width: number) {}
}
class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width)
  }
}
type Shape = Square | Rectangle

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // ... code
  }
}
```

This works because **class Rectangle** introduces both a type and a value, whereas **interface** only introduced a type.

## Type Operations Cannot Affect Runtime Values

```typescript
function asNumber(val: number | string): number {
  return val as number
}

// This will compile to the following JS:
function asNumber(val) {
  return val
}
```

The **as number** is a type operation, so it cannot affect the runtime behavior of your code. To check the value at 
runtime:

```typescript
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val
}
```

## Runtime Types May Not Be the Same as Declared Types

```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn()
      break
    case false:
      turnLightOff()
      break
    default:
      console.log(`I'm afraid I can't do that.`)
  }
}
```

The trap here is that **boolean** is a *declared* type and at runtime it goes away. The function doesn't narrow down the
possible values of "**value**" to **true** and **false**.

## You Cannot Overload a Function Based on TS Types

```typescript
function add(a: number, b: number) { return a + b }
function add(a: string, b: string) { return a + b } // Duplicate function implementation
```

TS provides a facility for overloading functions, but it operates entirely at the type level. 

> You can provide multiple declarations for a function but only a single implementation.

## Getting Comfortable with Structural Typing

Types in TS are "open", meaning if you have such interface:

```typescript
interface Vector3D {
  x: number
  y: number
  z: number
}
```

and a function that takes a parameter of this type:

```typescript
function calculateLength(v: Vector3D) {
  let length = 0
  for (const axis of Object.keys(v)) {
    const coord = v[axis] // Element implicitly has an 'any' type because 'string' can't be used to index type
    length += Math.abs(coord)
  }
  return length
}
```

Since **axis** is one of the keys of **v**, which is a **Vector3D**, it should be either "x", "y", or "z". And according
to the declaration of **Vector3D**, these are all **numbers**. BUT TS is correct to complain. The logic in the function
assumes that **Vector3D** is sealed and doesn't have other properties, but it could:

```typescript
const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway' }
calculateLength(vec3D) // OK, returns NaN
```
