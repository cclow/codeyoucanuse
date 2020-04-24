---
layout: post
title: Coding demo project and implementation of the API adaptor  (Hexagonal Architecture with Spring Boot — Part 2)
redirect_from:
  - /2020/04/23/hexagonal-architecture-using-spring-boot-part-2.html
---

This is the second article of a 4 part series explaining Hexagonal Architecture and how to implement such an architecture using Spring Boot and TDD.

[Part 1 - Introduction to Hexagonal Architecture and Key Concepts](/2020/04/23/hexagonal-architecture-part-1.html)

[Part 2 - Coding demo project and implementation of the API adaptor](/2020/04/23/hexagonal-architecture-part-2.html)

Part 3 - Implementation of Domain Services (in preparation)

Part 4 - Implementation of a MongoDB repository adaptor and REST adaptor to an external data source (in preparation)

*Update: The corresponding video is live on YouTube now*
<iframe width="560" height="315" src="https://www.youtube.com/embed/obd38-EM_KE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Spring Framework

Using the Spring Framework and its projects does not guarantee the hexagonal architecture or any good architecture. However, it is designed with the modularisation and dependency support consistent with hexagonal architecture.

Among other factors, its Dependency Injection and Inversion of Control patterns directly support the ports and adaptor architecture, and its integration capabilities accelerate adaptor implementation.

### Demo Application: Stock Position and Market Value Service

I'll demonstrate the development os an application how to implement a hexagonal architecture compliant application using Spring Boot.  We want to implement a service that returns how many shares a user has (stock position) and the current market value. This requires integration to both a data repository and a online source of stock prices.

The code can be found in [this Github repository](https://github.com/cclow/hexademo). First let's look at the pom.xml:

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 pom.xml %}

In this article, I'll describe the stages of coding and show the final code. There will be a screencast video to go through the step-by-step description of TDD and Hexagonal Architecture considerations in the process.

#### REST API Adapter

We'll use  TDD (Test Driven Development) for this project, so let's start with a test for the REST API.

What should the API adapter test cover? Let's list out the concerns of a API adapter.

* It should present an endpoint (e.g. "GET /stock-position") with optional selection predicates (e.g. "accept: application/json")
* It should  check for authorisation when required
* It should validate and convert the parameters in the request (which can come in path variables, query params, request body, header values, cookies, authentication, etc) into domain models and types where necessary
* It should call one or more domain service with the converted parameters
* It should convert the returned results from domain models into API DTOs (Data Transfer Object)
* It should return an appropriate HTTP status

![REST API Adapter](/images/REST-API-adapter.png)

In other words, the concerns of the API adapter includes everything between a request and the domain service for that request, but does not include any domain logic at all.

The fully fleshed out API test is:

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 GetStockPositionAndMarketValueApiTest.java %}

And the corresponding REST Controller

{% gist cclow/b1f1b72bc93aaee2f431cfd99620b801 StockPositionsController.java %}

Noted that the DTO for the response body, `GetStockPositionAndMarketValueApiResponseDto` has a long specific name. This is intentional as this DTO is specifically for this API endpoint. Unless clearly intended and specified in the API specs, *using the same DTO for different responses is unnecessary coupling*.

Continue to Part 3 (in preparation)
