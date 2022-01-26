# Keyof Type Operator

## The **keyof** type operator

The **keyof** operator takes an object type and produces a string or numerical literal union of its keys. The following
type `P` is the same as `"x" | "y"`.

```typescript
type Point = { x: number; y: number };
type P = keyof Point; // type P = keyof Point

type Arrayish = { [k: string]: boolean };
type A = keyof Arrayish; // type A = number

type Mapish = { [k: string]: boolean };
type M = keyof Mapish; // type M = string | number;
```

> [**NOTE**] **M** is `string | number` because JS object keys are always coerced to a string, so `obj[0]` is always
> the same as `obj["0"]`.