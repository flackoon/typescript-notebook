# The basics

- [Static type-checking](#static-type-checking)
- [Non-exception Failures](#non-exception-failures)
- [tsc, the TypeScript compiler](#tsc-the-typescript-compiler)
- [Emitting with Errors](#emitting-with-errors)
- [Explicit Types](#explicit-types)
- [Erased Types](#erased-types)
- [Downleveling](#downleveling)
- [Strictness](#strictness)
  - [noImplicitAny](#noimplicitany)
  - [strictNullChecks](#strictnullchecks)

## Static type-checking

_Static types systems_ describe the shapes and behaviors of what our values will be when we run our programs. A type 
checker like TypeScript uses that information and tells us when things might be going off the rails.


## Non-exception Failures

...runtime errors – cases where the JavaScript runtime tells us that it thinks something is nonsensical. Those cases come up
because the ECMAScript specification has explicit instructions on how the language should behave when it runs into
something unexpected.

For example, the spec says that trying to call something that isn't callable should throw an error. Maybe that sounds 
like "obvious behavior", but you could imagine that accessing a property that doesn't exist on an object should throw 
and error too. Instead, JS behaves differently and returns `undefined`.

Ultimately, a static type system has to make the call over what code should be flagged as an error in its system, even
if it's "valid" JS that won't immediately throw an error.

While sometimes that implies a trade-off in what you can express, the intent is to catch legitimate bugs in our programs.
And TypeScript catches _a lot_ of legitimate bugs.


## tsc, the TypeScript compiler

To compile a TS file, run

`$ tsc filename.ts`

This will generate a JS file after tsc _compiles_ or _transforms_ it into a plain JavaScript file. And if we check the
contents, we'll see what TypeScript spits out after it processes a .ts file. 

The compiler tries to emit clean readable code that looks like something a person would write. While that's not always so
easy, TypeScript indents consistently, is mindful of when our code spans across different lines of code, and tries to
keep comments around.

If we intentionally make an error, say an argument one, we will get an error message and a stacktrace after compiling.  


## Emitting with Errors

If you compile a file that will make TS raise errors, you will still get it compiled to a .js file. That is because one
of TypeScript's code values is: much of the time, _you_ will know better than TypeScript.

To reiterate from earlier, type-checking code limits the sorts of programs you can run, and so there's a tradeoff on 
what sorts of things a type-checker finds acceptable. Most of the time that's okay, but there are scenarios where those 
checks get in the way. For example, when migrating JS code over to TS and introducing type-checking errors. Eventually
you'll get around to cleaning things up for the type-checker, but the original JS code was already working! Why should
converting it over to TS stop you from running it?

So TS doesn't get in your way. Of course, over time, you may want to be a bit more defensive against mistakes, and make TS
act a bit more strictly. In that case, you can use the `noEmitOnError` compiler option.

`tsc --noOmitOnError filename.ts`


## Explicit Types

We don't always have to write explicit type annotations. In many cases, TS can even just _infer_ the types for us even if
we omit them.

```typescript
let msg = "hello there!";
//   ^
//   let msg: string     What the editor would show if you hover over the word
```

This is a feature, and it's best not to add annotations when the type system would end up inferring the same type anyway.


## Erased Types

Type annotations aren't part of JS (or ES to be pedantic), so there really aren't any browser or other runtimes that can
just run TypeScript unmodified. That's why TS needs a compiler in the first place – it needs some way to strip out or 
transform any TS-specific code so that you can run it. Most TS-specific code gets erased away.

> Type annotations never change the runtime behavior of your program.


## Downleveling

Upon compilation, template strings get transformed to simple string concatenation.

This happens because they are a feature from a version of ES. TS has the ability to rewrite code from newer versions of
ES to older ones such as ES3 or ES5. This process of moving from a newer or "higher" version of ES down to an older or
"lower" one is sometimes called _downleveling_.

By default TS targets ES3, an extremely old version of ES. The targeted ES version can be specified by using the `target`
option. Running with `--target es2015` changes TS to target ES2015, meaning code should be run whenever ES2015 is 
supported. 

> While the default target is ES3, the great majority of current browser support ES2015. Most developers can therefore
> safely specify ES2015 or above as a target, unless compatibility with certain ancient browsers is important.


## Strictness

Per default, the experience with TS is more loose, where types are optional, inference takes the most lenient types, and
there's no checking for potentially `null`/`undefined` values. Much like how `tsc` emits in the face of errors, these 
defaults are put in place to stay out of your way. If you're migrating existing JS, that might be a desirable first step.

There are strictness settings, however, that turn static type-checking from a switch (either your code is checked or
not) into something closer to a dial. The stricter the type-checking, the bigger the possibility of required extra work,
but generally speaking it pays for itself in the long run, and enables more thorough checks and more accurate tooling.
When possible, a new codebase should always turn these strictness checks on.

The `strict` flag in the CLI, or `"strict": true` in a **tsconfig.json** toggles them all on simultaneously, but we
can opt out of them individually. The two biggest ones you should know about are `noImplicitAny` and `strictNullChecks`.

### noImplicitAny

In some places, TS doesn't try to infer types and instead falls back to the most lenient one – `any`. This isn't the worst
thing that can happen – after all, falling back to any is just the plain JS experience anyway. 
However, using `any` often defeats the purpose of using TS in the first place. Turning on the `noImplicitAny` flag will 
issue an error on any variables whose type is implicitly inferred as `any`.

### strictNullChecks

By default, values like `null` and `undefined` are assignable to any other type. This can make writing code easier, but
forgetting to handle `null` and `undefined` is the cause of countless bugs in the world. The `strictNullChecks` flags 
makes handling `null` and `undefined` more explicit, and _spares_ us from worrying about whether we _forgot_ to handle
`null` and `undefined`.
