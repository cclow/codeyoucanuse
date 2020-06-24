---
layout: post
title: Writing Better Code with Reactor 3 - Part 2
---
### Hot and Cold Publishers, ConnectableFlux 
Broadly speaking, there are 2 types of publishers: 

1. Cold publishers which starts anew for each subscriber. 
Examples of cold publishers include requests to external web apis and to databases

![Cold Publishers](/images/cold-publishers.png)

2. And Hot publishers, the data is generated independently and shared between subscribers.
An example of a hot publisher is a flux listening on a mongodb change stream.

![Hot Publishers](/images/hot-publishers.png)

The simplest hot publisher can be generated using the `just()` factory method.
Let's write some code to explore this.
We start by timestamping the start of execution, and the creation of the data for the Mono with the following code.
A delay is added before the subscription so we could observe whether the data is created at *assembly* or *subscription* time.

```java
public class Part2Test {
    @Test
    void hotCold() throws InterruptedException {
        System.out.println("START: " + currentInstant());
        Mono<Long> mono = Mono.just(generateData());

        Thread.sleep(500);
        mono.subscribe();
        
        Thread.sleep(500);
        mono.subscribe();
    }

    private Long generateData() {
        System.out.println("Generated data at " + currentInstant());
        return 0L;
    }

    private long currentInstant() {
        return Instant.now().toEpochMilli() % 10000;
    }
}
```
Running this test gives the following output:
```
START: 6484
Generated data at 6484

Process finished with exit code 0
```

You can see that the data is generated only once, just after the start of execution, at the creation of the publisher, and not at its subscription.

This applies to the `Flux.just()` method as well.

The simplest way to convert this to a cold publisher is to use the `defer()` method which takes a publisher supplier as parameter.
Changing the creation of the Mono in the previous code example as shown:

```java
  Mono<Long> mono = Mono.defer(() -> Mono.just(generateData()));
```

we now got an output similar to the following which shows that the data are generated for each subscription when subscribed. 

```
START: 4862
Generated data at 5419
Generated data at 5925
```

In contrast, the `Flux.interval()` is a cold publisher. Using the following code, we can see from the output that each subscription is independently started and executed.

```java
  @Test
  void hotCold() throws InterruptedException {
      System.out.println("START: " + currentInstant());
      Flux<Long> interval = Flux.interval(Duration.ofMillis(50));

      Thread.sleep(200);
      interval.subscribe(n -> System.out.println("1: at " + currentInstant() + ": " + n));

      Thread.sleep(200);
      interval.subscribe(n -> System.out.println("2: at " + currentInstant() + ": " + n));
      Thread.sleep(200);
  }
```
Output:
```
START: 7726
1: at 8087: 0
1: at 8138: 1
1: at 8186: 2
1: at 8237: 3
1: at 8289: 4
2: at 8289: 0
1: at 8338: 5
2: at 8338: 1
1: at 8386: 6
2: at 8387: 2
2: at 8437: 3
1: at 8438: 7
```

One way to convert a cold Flux to a hot one is using the `share()` method as shown:
```java
        Flux<Long> interval = Flux.interval(Duration.ofMillis(50)).share();
```
The resulting output shows that the two subscriptions are sharing the same data stream:
```
START: 1135
1: at 1502: 0
1: at 1550: 1
1: at 1602: 2
1: at 1651: 3
1: at 1700: 4
2: at 1700: 4
1: at 1752: 5
2: at 1752: 5
...
```

If we want the second (or any subsequent) subscription to get the data elements from the start, we can use the `cache()` method.
Adding it to the previous example,
```java
        Flux<Long> interval = Flux.interval(Duration.ofMillis(50)).share().cache();
```
We get the following output which shows that the previous elements are received by the new subscription.
```
START: 8451
1: at 8827: 0
1: at 8875: 1
1: at 8921: 2
1: at 8971: 3
2: at 8976: 0
2: at 8976: 1
2: at 8976: 2
2: at 8976: 3
1: at 9020: 4
2: at 9020: 4
...
...
``` 

We can also customise `cache()` with parameters for maximum cache history and duration.
Note that cache() should be called after share() for it to work as expected.

Without the `cache()`, the publisher is dropped if there are no active subscription. From the example above, we remove the `cache()`, and change the first subscription to take only the first 2 elements so that it ends before the second subscription starts.

```java
  interval
      .take(2)
      .subscribe(n -> System.out.println("1: at " + currentInstant() + ": " + n));
```
We can see from the output that the Flux is restarted.

```
START: 820
1: at 1189: 0
1: at 1233: 1
2: at 1334: 0
2: at 1334: 1
2: at 1334: 2
2: at 1334: 3
...
```

If we `take(5)` instead so that the second subscription starts before the first completes, the Flux does not restart.

Another way to turn a cold publisher to a hot one is through the `publish()` method which returns a ConnectableFlux.
As is, a ConnectableFlux is not started by subscriptions.

Calling `refCount()` on a `ConnectableFlux` returns a shared flux, so `publish().refCount()` is equivalent to call the `share()` method.
Like `share()`, the data stream is restarted when subscriber count falls to zero.

We can specify a minimum number of subscribers, so `refCount(2)` will only start on the second subscription.
Using the following code:

```java
  Flux<Long> interval = Flux.interval(Duration.ofMillis(50)).publish().refCount(2);
  // ...
  interval
      .take(2)
      .subscribe(n -> System.out.println("1: at " + currentInstant() + ": " + n));
```
we observe from the output that the data stream starts at the second subscription but does not stop when the first ends.

```
START: 7878
1: at 8446: 0
2: at 8446: 0
1: at 8496: 1
2: at 8496: 1
2: at 8546: 2
2: at 8596: 3
```

We can also specify a grace period so the data stream waits before disconnecting when the subscriber count falls to zero.

We can also start the `ConnectableFlux` using the `connect()` method.
With the following code:

```java
  ConnectableFlux<Long> interval = Flux.interval(Duration.ofMillis(50)).publish();
  interval.connect();

  Thread.sleep(200);
  interval.subscribe(n -> System.out.println("1: at " + currentInstant() + ": " + n));

  Thread.sleep(200);
  interval.subscribe(n -> System.out.println("2: at " + currentInstant() + ": " + n));
```
We observe from the output that the data stream starts at the `connect()` call, even before the first subscription.
```
START: 6030
1: at 6367: 4
1: at 6416: 5
1: at 6465: 6
1: at 6514: 7
1: at 6568: 8
2: at 6568: 8
...
```

We can also make the connection dependent on subscribers using the `autoConnect()` method.
Note that the subscribers have to subscribe to the resulting Flux, and not the ConnectableFlux for this to work.

```java
  Flux<Long> interval = Flux.interval(Duration.ofMillis(50)).publish().autoConnect();
```
Output:
```
START: 3767
1: at 4128: 0
1: at 4176: 1
1: at 4222: 2
1: at 4276: 3
1: at 4322: 4
2: at 4322: 4
1: at 4374: 5
2: at 4374: 5
...
```

Like `refCount()`, you can specify a minimum subscriber count for `autoConnect()`. Note that once connected, it does not disconnect even when the last subscriber leaves

You can also watch a screencast of this tutorial.
<iframe width="560" height="315" src="https://www.youtube.com/embed/iT2zQ1jcqoA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
