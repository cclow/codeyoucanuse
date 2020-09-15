---
layout: post
title: Merge Paged Results with Reactor 3
---

## The Scenario

Large data sets are often only available through paginated APIs.
To retrieve multi-paged results from these APIs, we have to make multiple calls and merge the results.

This is usually quite straight forward with imperative programming style.
In our scenario, we want to retrieve the entire data set,
and while the response page size is documented,
we chose not to hardcode it since this might change.


```java
skip = 0
do {
    pagedResultList = getNextBatch(skip)
    lastCount = pagedResultList.size
    if (lastCount > 0) {
        concatOrProcessResult(pagedResult)
        skip += lastCount
    }
} while (lastCount > 0)
```

In functional or reactive programming style, the code is not as obvious.
Particularly troublesome are the `skip` variable and the single threaded looping required.

## First Solution

The first solution we tried was using the `Flux.generate()` method.
The `generate()` method allows us to create a stateful `Flux` synchronously
by emitting each element individually.

The code is similar to the imperative `do-while` loop:

```java
Flux.generate(
    () -> 0L, // initial state provider, in this case the skip variable
    (Long skip, SynchronousSink<List<Result>> sink) -> { // state = skip
      List<Result> nextBatch = getNextBatch(skip)
          .block(Duration.ofSeconds(2));        // BLOCKING!
      if (nextBatch != null && nextBatch.isEmpty()) {
        sink.complete();
      } else {
        sink.next(nextBatch);
      }
      return skip + nextBatch.size();
    }
).flatMapIterable(list -> list);
```

Unfortunately this solution requires blocking for the page result list.

## Second Solution

The second solution is to use the `repeatWhen()` method combined with
`Flux.deferWithContext()`.

When an upstream `Publisher` completes,
`repeatWhen()` calls a handler with two arguments.
The first is the count of elements from the just completed subscription.
The second is a companion synchronous sink of `Context`.

When a `Context` element is emitted on this companion sink,
the element is merged into the context of the upstream `Publisher` 
and it is re-subscribed.

We use the context to keep track of the `skip` variable.   

```java
Flux.deferWithContext(
    context -> {
      Long skip = context.getOrDefault("skip", 0L);
      return getNextBatch(skip)
          .flatMapIterable(list -> list);
    })
.repeatWhen(repeatHandler -> repeatHandler.handle((count, sink) -> {
  if (count > 0) { // number of elements emitted during last subscription
    Long skip = sink.currentContext().getOrDefault("skip", 0L);
    sink.next(Context.of("skip", skip + count));
  } else {
    sink.complete();
  }
}));
```

This approach is non-blocking.

## Conclusion

When programming in the reactive style,
concerns such as synchronicity, blocking, and scheduling must be considered.
Most reactive frameworks provide a plethora of methods to cater to these consideration, but the method choice is complicated due to a number of reasons.

Most of these concerns can be difficult to understand.
The methods tend to hide the implementation,
and one can argue that even reading the code would not help.

Making matters worse, there are a very large number of methods.
Just for the the `Publishers` (`Mono` and `Flux`),
there are more than 200 methods.

These are often named and documented inconsistently,
complicating the choice of a method for a specific task short of understanding the whole list of methods.

The combination of `repeatWhen()` and `Flux.deferWithContext()` helps us to simulate a loop reactively,
retrieving a large data set through an API that returns paginated results.
