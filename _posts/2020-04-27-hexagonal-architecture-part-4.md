---
layout: post
title: Implementation of a MongoDB repository adaptor  (Hexagonal Architecture with Spring Boot — Part 4)
---

This is the fourth article of a <s> 4 </s> 5 part series explaining Hexagonal Architecture and how to implement such an architecture using Spring Boot and TDD.

[Part 1 - Introduction to Hexagonal Architecture and Key Concepts](/2020/04/23/hexagonal-architecture-part-1.html)

[Part 2 - Coding demo project and implementation of the API adaptor](/2020/04/23/hexagonal-architecture-part-2.html)

[Part 3 - Implementation of Domain Services](/2020/04/27/hexagonal-architecture-part-3.html)

[Part 4 - Implementation of MongoDB repository adaptor](/2020/04/27/hexagonal-architecture-part-4.html)

[Part 5 - Implementation of REST adaptor to an external data source](/2020/04/27/hexagonal-architecture-part-5.html)

### Data Repository using MongoDB

Next we'll implement the data repository required by domain service `GetStockPositionService`, as specified by the interface `StockPositionsRepository`.

Based on the expected behaviour of this interface, we specify two tests, the first for when the stock position is present, and the second for when it is not.

Notice that the interface and its tests are in the domain service package. This is because the behaviour of this interface *is independent of the implementation*.

However, to pass the tests, we need an implementation of this interface. Since the required behaviour is quite straightforward, we have chosen using Spring Data's `ReactiveMongoRepository` for this implementation, though the tests `StockPositionsRepositoryIntegrationTest` applies even if we use some other implementation, for instance, using the `ReactiveMongoTemplate` or some other data storage technologies.

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 StockPositionsRepositoryIntegrationTest.java %}

A few key points to note:

1. In the domain repository interface (`StockPositionsRepository`) I have chosen to use the equivalent method names, e.g. `findOneByUserAndSymbol`, as specified by `ReactiveMongoRepsotiory`. While this simplifies the code, it does not impose a dependency on the implementation since we could just as easily implement the repository some other way.
2. We could have extended `StockPositionsRepository` directly from  `ReactiveMongoRepository` without using `ReactiveMongoStockPositionRepository` but that would mean that the domain interface, `StockPositionsRepository`, would now have knowledge about the implementation and be dependent on the adaptor implementation.
3. Instead of using the domain model, `StockPosition`, directly as the document specification, `StockPositionDocument` which extends `StockPosition` is used as the document class in `ReactiveMongoRepository`. This, again, insulates the domain model from the implementation details, e.g. `@Id ObjectId id` .

It is debatable if such pedantry, especially points 2 & 3, simplifies or complicates the code, but I've chosen to use them here as illustrations of the options.

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 ReactiveMongoStockPositionRepository.java %}

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 StockPositionDocument.java %}

[Back to Part 3](/2020/04/27/hexagonal-architecture-part-3.html)

[Continue to Part 5](/2020/04/27/hexagonal-architecture-part-5.html)
