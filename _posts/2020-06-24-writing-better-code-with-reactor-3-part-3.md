---
layout: post
title: Writing Better Code with Reactor 3 - Part 3
---
### Basic Reactive Testing with StepVerifier

In this article, we’ll explore testing of reactive data streams with the `StepVerifier`.

Let's start by creating an empty Mono with `Mono.empty()`.
This mono completes before emitting any data element.

We then create a StepVerifier from this empty mono, and we expect this to complete without any data element using `expectComplete`, and end the test with the `verify()` method.

```java
   Mono<Object> empty = Mono.empty();

   StepVerifier.create(empty)
       .expectComplete()
       .verify();
```
It is very important to call verify() or one of its variants. If we remove it, the test now passes despite nothing else changing.
It would seem to runs and pass, but the expectations are not executed.

`expectComplete()` and `verify()` are often called sequentially and we could replace them with just `verifyComplete()`.

```java
   // equivalent to above
   StepVerifier.create(empty)
       .verifyComplete();
```

If instead of an empty stream, we expect the publisher to produce a data element before completing, we can add this expectation with `expectNext()`.
This would fail the test since the `empty()` publisher does not produce any data elements.

```java
  Mono<Integer> mono = Mono.just(0);

  StepVerifier.create(mono)
      .expectNext(0)
      .verifyComplete();
```

`expectNext()` can take a data sequence as expectation.
Alternatively, we could split the expectation into individual expectations.
Note that the order of the sequence is important. If we we reverse the order of the expectations, the test will fail.
Also, all data elements have to be accounted for, if we add or remove an expectation, the test will fail.

```java
  Flux<Integer> flux = Flux.just(0, 2, 4);

  StepVerifier.create(flux)
      .expectNext(0, 2, 4)
      .verifyComplete();

  // requivalent
  StepVerifier.create(flux)
      .expectNext(0)
      .expectNext(2)
      .expectNext(4)
      .verifyComplete();
```

We can also use expectNextSequence() to match the data items to an Iterable such as a List.
```java
  StepVerifier.create(flux)
      .expectNextSequence(Arrays.asList(0, 2, 4))
      .verifyComplete();
```

Besides expecting exact values, we can also verify the next item with matchers using `expectNextMatches()`.

We can use `assertNext()`, giving it an assertConsumer.
We can also use the `consumeNextWith()` though this need not be an assertion.

```java
  StepVerifier.create(flux)
      .assertNext(n -> Assertions.assertTrue(n == 0))
      .consumeNextWith(n -> System.out.println("n = " + n))
      .expectNextMatches(n -> n % 2 == 0)
      .verifyComplete();
```

If we're interested only in how many data elements are emitted, we could use `expectNextCount()`.
```java
  StepVerifier.create(flux)
      .expectNextCount(3)
      .verifyComplete();
```

We can use `thenConsumeWhile()` method to consume data items as long as a condition is met, and also provide a consumer to process it if necessary.

```java
  StepVerifier.create(flux)
      .thenConsumeWhile(n -> n <= 2)
      .consumeNextWith(n -> Assertions.assertTrue(n % 2 == 0))
      .verifyComplete();
```

Sometimes, we don’t need to test the rest of the data stream, we can use `thenCancel()` to cancel data streaming.
There will not be a complete event when we cancel so we can use `verify()` instead. 

```java
  StepVerifier.create(flux)
      .thenConsumeWhile(n -> n <= 2)
      .thenCancel()
      .verify();
```

An alternative way to create the StepVerifier is using the as() method of the publisher.
This fluent style is often more readable and preferred.

```java
  flux.as(StepVerifier::create)
      .thenConsumeWhile(n -> n <= 2)
      .thenCancel()
      .verify();
```
How about error events?
We can add an expectation of error using expectError(), 
to specify the type of error we expect, the expected error message
or using expectErrorMatches() to do both.
```java
  Mono.error(new IllegalAccessError("boom"))
      .as(StepVerifier::create)
      // .expectError(IllegalAccessError.class)
      // .expectError("boom")
      .expectErrorMatches(e -> e instanceof IllegalAccessError && e.getMessage().equals("boom"))
      .verify();
```

Sometimes we might want to test publishers that have a long running time.
For example `Mono.delay()` can be used to emit a 0 after a duration, say one minute.
Instead of waiting a minute in the test for this to complete, we could use the `withVirtualTime()` method to shorten the testing time.

Note that the tested publisher must be created within the `withVirtualTime()` method, so instead of passing in a publisher directly, you need to provide a publisher supplier.
Also, creating the publisher outside, and then simply returning it from the supplier *does not* work.
You have to move the creation of the publisher under test inside the `withVirtualTime()` method.

In addition, we need to call the `thenAwait()` method to fast forward the test.
While you could call `thenAwait()` without a duration, its behaviour seems to be more consistent with a specified duration.
This enables us to skip ahead in time and examine all the events happening during that period.

```java
  StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofMinutes(1)))
      .thenAwait(Duration.ofMinutes(1))
      .expectNext(0L)
      .verifyComplete();
```

Instead of the `thenAwait()` method, we can also use the `expectNoEvent()` method.
Besides fast forwarding, the expectNoEvent() method also adds an expectation that no events occur during the wait.

Be aware that a subscription is an event, so expectNoEvent on a new subscription will fail the test unless we consume it with an `expectSubscription()` first.
For the same reason, expectNoEvent is deprecated as the first verification step.

`expectNoEvent()`'s interaction with the `thenAwait()` method can be confusing.
If you call `thenAwait()` before `expectNoEvent()`, the `expectNoEvent()` duration starts after thenAwait() duration.
Also if the await period is too short, not all events would had been generated.

As a general rule, thenAwait should come after expectNoEvent, and the total expectNoEvent plus thenAwait time must be sufficient to cover subsequent events.

```java
  StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofMinutes(1)))
      .expectSubscription()
      .expectNoEvent(Duration.ofMinutes(1))
      .expectNext(0L)
      .verifyComplete();
```

The `verify()` method returns the Duration of the test execution.
 
You can also use `verifyAndAssertThat()`, with `tookMoreThan()` and `tookLessThan()` assertions for the duration of the test.
Note that this is the duration of the *test*, not the data stream, which is different especially when we use `withVirtualTime()`.

```java
  StepVerifier.withVirtualTime(() -> Mono.delay(Duration.ofMinutes(1)))
      .thenAwait(Duration.ofMinutes(1))
      .expectNext(0L)
      .expectComplete()
      .verifyThenAssertThat()
      .tookLessThan(Duration.ofMinutes(1));
```

`verifyThenAssertThat()` also allows us to bring up other StepVerifier Assertions, mostly for dropped or discarded data elements or errors.

You can also watch a screencast of this tutorial.
<iframe width="560" height="315" src="https://www.youtube.com/embed/RsoezWdNtDs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
