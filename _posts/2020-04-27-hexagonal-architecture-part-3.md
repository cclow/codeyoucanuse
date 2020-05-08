---
layout: post
title: Implementing the Domain Services (Hexagonal Architecture with Spring Boot — Part 3)
---

This is the third article of a <s> 4 </s> 5 part series explaining Hexagonal Architecture and how to implement such an architecture using Spring Boot and TDD.

[Part 1 - Introduction to Hexagonal Architecture and Key Concepts](/2020/04/23/hexagonal-architecture-part-1.html)

[Part 2 - Coding demo project and implementation of the API adaptor](/2020/04/23/hexagonal-architecture-part-2.html)

[Part 3 - Implementation of Domain Services](/2020/04/27/hexagonal-architecture-part-3.html)

[Part 4 - Implementation of MongoDB repository adaptor](/2020/04/27/hexagonal-architecture-part-4.html)

[Part 5 - Implementation of REST adaptor to an external data source](/2020/04/27/hexagonal-architecture-part-5.html)

Part 2 of the video is live now:
<iframe width="560" height="315" src="https://www.youtube.com/embed/NnGuWh-jDAc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Domain Services

At the end of implementing the REST API adaptor, we have mocked out two domain services `GetStockPositionService` and `GetStockMarketValueService`.

As you go through the code below, notice that we will write the domain services functionally, and we can test them without using the Spring framework for dependency injection. Testing domain services functionally and without the Spring framework enables very fast testing.

Also, notice that where the domain services need to call some adaptors, whether data repository or external services, a port to interface is used. Again, in the tests, we specify the required behaviour of these adaptors through these ports, and the domain services does not have to know any further details of these adaptors beyond their behaviours.

Let's start with the test for `GetStockPositionService`, `GetStockPositionServiceTest` as shown here:

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockPositionServiceTest.java %}

The repository required for this domain service is specified as an interface and provided directly to the service as a constructor parameter.

{% gist  cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockPositionService.java %}

The other domain service `GetStockMarketValueService` and its test `GetStockMarketValueServiceTest` are written in a similar style.

{% gist  cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockMarketValueServiceTest.java %}

{% gist  cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockMarketValueService.java %}

`GetStockMarketValueService` requires a source for the current market price of the stock for calculating its result. For now, we'll specify an port as an interface `GetStockMarketPricePort` for this service.

[Back to Part 2](/2020/04/23/hexagonal-architecture-part-2.html)

[Continue to part 4](/2020/04/27/hexagonal-architecture-part-4.html)
