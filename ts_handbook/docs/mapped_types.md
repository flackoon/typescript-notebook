# Mapped Types

Mapped types build on the syntax for index signatures, which are used to declare the types of properties which have not
been declared ahead of time:


```typescript
type TypeA = {
  [key: string]: boolean | Horse;
}
```

A mapped type is a generic type which uses a union of **PropertyKeys** (frequently created via a `keyof`) to iterate
keys to create a type:

```typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
}
```

In this example, **OptionsFlags** will take all the properties from the type **Type** and change their values to be a
boolean. 

```typescript
type FeatureFlags = {
  darkmode: () => void;
  newUserProfile: () => void;
}

type FeatureOptions = OptionsFlags<FeatureFlags>;
// type FeatureOptions = {
//   darkMode: boolean;
//   newUserProfile: boolean;
// }
```


## Mapping Modifiers

There are two additional modifiers which can be applied during mapping: **readonly** and `?` which affect mutability and
optionally respectively. 

You can remove or add these modifiers by prefixing with `-` or `+`. If you don't add a prefix, then `+` is assumed.

```typescript
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlokedAccount = CreateMutable<LockedAccount>;
// type UnlockedAccount = {
//   id: string;
//   name: string;
// };

// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
// type User = {
//   id: string;
//   name: string;
// };
```

## Key Remapping via `as`

You can re-map keys in mapped types with an `as` clause in a mapped type:

```typescript
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

You can leverage features like template literal types to create new property names from prior ones:

```typescript
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
  name: string;
  age: number;
  location: string;
}

type LazyPerson = Getters<Person>;
// type LazyPerson = {
//   getName: () => string;
//   getAge: () => number;
//   getLocation: () => string;
// }
```

You can filter out keys by producing never via a conditional type:

```typescript
// Remove the 'kind' property
type RemoveKindField<Type> = {
  [Propety in keyof Type as Exclude<Property, "kind">]: Type[Property]
}
```

You can map over arbitrary unions, not just unions of `string | number | symbol`, but unions of any type:

```typescript
type EventConfig<Events extends { kind: string }> => {
  [E in Events as E["kind"]: (event: E) => void;
}

“type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
 
type Config = EventConfig<SquareEvent | CircleEvent
// type Config = {
//   square: (event: SquareEvent) => void;
//   circle: (event: CircleEvent) => void;
// };
```
