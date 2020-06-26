---
layout: post
title: Writing Better Code with Reactor 3 - Part 4
---
### Mapper Operators

In this screencast, we explore operators, or methods that connect upstream publishers with downstream subscribers, potentially through one or more other operators.
In our analogy of the reactive data stream as an assembly line, the operator could be thought of as stations along the line for transforming, combining, and regulating the flow of parts and assembled objects.

There are over 100 operator methods in [Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) and [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) for manipulating reactive data streams.
We’ll focus on the mappers which transforms upstream data elements into one or more downstream elements.

Operators covered: 
- `map`
- `flatMap`, `flatMapMany`, `flatMapSequential`, `concatMap`
- `switchMap`
- `flatMapIterable`, `concatMapIterable`
- `flatMapDelayError`, `flatMapSequentialDelayError`, `concatMapDelayError`
- `expand`, `expandDeep`
- `handle`

<iframe width="560" height="315" src="https://www.youtube.com/embed/FmXJsxvzwA4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
