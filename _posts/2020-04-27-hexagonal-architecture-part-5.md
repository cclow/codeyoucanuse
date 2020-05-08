---
layout: post
title: Implementation of REST adaptor to an external data source  (Hexagonal Architecture with Spring Boot — Part 5)
---

This is the fourth article of a <s> 4 </s> 5 part series explaining Hexagonal Architecture and how to implement such an architecture using Spring Boot and TDD.

[Part 1 - Introduction to Hexagonal Architecture and Key Concepts](/2020/04/23/hexagonal-architecture-part-1.html)

[Part 2 - Coding demo project and implementation of the API adaptor](/2020/04/23/hexagonal-architecture-part-2.html)

[Part 3 - Implementation of Domain Services](/2020/04/27/hexagonal-architecture-part-3.html)

[Part 4 - Implementation of MongoDB repository adaptor](/2020/04/27/hexagonal-architecture-part-4.html)

[Part 5 - Implementation of REST adaptor to an external data source](/2020/04/27/hexagonal-architecture-part-5.html)

Part 2 of the video is live now:
<iframe width="560" height="315" src="https://www.youtube.com/embed/NnGuWh-jDAc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### REST Adapter for Stock Market Price

To complete the service, the last interface to implement is `GetStockMarketPricePort` for retrieving the market price of the stock. We will use the free service form [AlphaVantage](https://www.alphavantage.co/documentation/) for this.

![alphavantage](/images/alphavantage.png)

There are several providers with different service levels and pricing that you could close from, and you can choose any and implement the corresponding adapter, *without changing any other parts of the source code*. Herein lies the power of the hexagonal architecture to modularise the code.

We start by choosing a suitable API endpoint to provide the required data, in this case since we only need the last closing price, we can use the `TIME_SERIES_DAILY` endpoint.

We'll need a JSON mapper for the data returned by this endpoint, so let's download a sample and place it into the test resource `test/respurces/alphavantage-samples/time-series-daily.json`. 

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 time-series-daily.json %}

We then create a test `AlphaVantageTimeSeriesDailyJsonTest` to make sure the mapping classes are correct. 

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 AlphaVantageTimeSeriesDailyJsonTest.java %}

The resulting classes for the JSON mapping are

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 AlphaVantageTimeSeriesDailyJson.java %}

With that out of the way, we can now implement the request to AlphaVantage. Again, let's start with the test. Like the repository test, *this test is implementation independent*. It should specify the expected behaviour of the interface `GetStockMarketPricePort` regardless of what the implementation is.

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockMarketPricePortIntegrationTest.java %}

The implementation is quite straight forward using this JSON mapper, and we'll elaborate on one detail.

To use the service, we need an API key (sometimes called authentication token) from AlphaVantage. We want to keep this API key a secret so not should not be in the source code. To get around this, we'll set the environment variable `ALPHAVANTAGE_API_KEY` to the value of this key.

We then use `@Value("${alphavantage.api.key}")` to retrieve this key in the code. Note that Spring automatically converts the environment variable `ALPHAVANTAGE_API_KEY` to match the `alphavantage.api.key` property.

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 AlphaVantageGetStockMarketPrice.java %}

### End-to-End Test

We have now implemented all the adapters and domain services, does this work when integrated end-to-end? 

Let's write an end-to-end test to confirm this. We'll use the `@SpringBootTest` annotation to link everything together.

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockPositionAndMarketValueApiE2ETest.java %}

### Conclusion

In this series, we explain the key concepts of the Hexagonal Architecture, and demonstrated how to implement it using TDD in Spring Boot.

You can download the full source code is in [this Github repository](https://github.com/cclow/hexademo) to try it out.

There's also a two part screencast to walk through the development process. Screencast 1 is here
<iframe width="560" height="315" src="https://www.youtube.com/embed/obd38-EM_KE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Screencast 2
<iframe width="560" height="315" src="https://www.youtube.com/embed/NnGuWh-jDAc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

[Back to Part 4](/2020/04/27/hexagonal-architecture-part-4.html)
