# Type Design

## Prefer types that always represent valid states

A key to effective type design is crafting types that can only represent a valid state. This way your code will be
easier to write and TS will have an easier time checking it.

## Be liberal in what you accept and strict in what you produce

This idea is known as the *robustness principle* or *Postel's Law*, after Jon Postel, who wrote it in the context of TCP:

> *TCP implementations should follow a general principle of robustness: be conservative in what you do, be liberal in
> what you accept from others.*

A similar rule applies to the contracts for functions.


## Don't repeat type information in documentation

