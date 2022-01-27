# Typeof Type Operator

TS adds a **typeof** operator so yo can use it in a *type* context  to refer to the *type* of a variable or property:

```typescript
let s = "hello";
let n: typeof s;
// let n: string
```

This isn't very useful for basic types, but combined with other type operators you can use **typeof** to conveniently
express many patterns. For example, let's start by looking at the predefined type **ReturnType<T>**. It takes a *function
type* and produces its return type:

```typescript
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
// type K = boolean
```

## Limitations

TS intentionally limits the sorts of expressions you can **typeof** on.
Specifically it's only legal to use **typeof** on identities (i.e. variable names) and their properties. This helps
avoiding the confusing trap of writing code you think is executing, but isn't.
