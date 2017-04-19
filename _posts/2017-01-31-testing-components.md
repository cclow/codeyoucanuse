---
layout: post
title: Testing Angular 2+ Components The Right Way
---

## Why Most Angular 2+ Testing Tuorials Are Wrong

Most Angular 2+ testing tutorials (including the official tutorial on angular.io) recommend
the wrong way to test Components.

Consider the following component:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e sut.component.ts %}

and its test code:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e sut.component.spec.ts %}

The test code reaches into the component to modify
the value of `title`.
This is a common practice you'll find in most tutorials, 
and it is problematic in a couple of ways.

#### Problem 1: The test is fragile, and can break even when the implementation is correct.

This test makes assumptions into the implementation of `SutComponent`,
i.e. it assumes that the component has a field called `title`.
However, `SutComponent` was intended to have a property `title`
that a parent could bind to,
and not a public field named `title`.

If for some reason we need the field to be `someOtherName`,
but the property name remains as `title`, as illustrated below:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e prob-1.ts %}

This implementation would still work correctly in the application
but the test would now fail.

#### Problem 2: The test is incomplete, it can pass even if implementation is broken.

For the same reason, the original test does not exercise the property binding
`title`.

If we remove the property binding but keep the field `title`,
as illustrated in the code below,
the component would now not work as intended,
but the test above would still pass and not catch the error.

{% gist cclow/44ac8952fcc2416197f62ae963c9797e prob-2.ts %}

### The Right Way

To test Angular 2+ Components correctly,
our tests must be written *without* knowledge or assumption
of the component's implementation details.
Tests must be written to test a component's public interfaces and contracts and *only* 
its public interfaces and contracts.

What are the public interfaces of Angular 2+ Components?

A well-encapsulated component can have the following types of public interfaces:
* Property binding, as illustrated above
* Event bindings, through which a component can emit an event to its parent
* UI rendering and interactions
* Interactions with models, business logic, and external systems through 
dependency-injected services

These must be the only interfaces used in the Angular 2+ component test code.

Consider the following component `ArticleComponent` that contains
all these types of public interfaces:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.ts %}

~~You can see that we declare all the fields and methods as `private`.~~
~~This would not affect the application, and it helps enforcing the strict encapsulation during development.~~ (Update 2017-04-19: Private fields are no longer accepted for production build. However, to preserve modularity, fields should not be accessed directly from outside its component.)

To test `ArticleComponent`, we start by creating a mock parent component that
binds to the properties and events of `ArticleComponent`. 
We also create a mock instance of `ArticlesService`,
`mockArticlesService`, with the interfaces we intend to use in the component:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.0.ts %}

Instead of instantiating `ArticleComponent`,
we instantiate the `TestParent` component which 
binds to the properties and events of `ArticleComponent`.
The `ArticleComponent` test fixture is then extracted from the `TestParent` fixture.

To test the interactions between `ArticleComponent` and `ArticlesService`,
`mockArticlesService` is injected as a provider for `ArticlesService`.
We then spy on this mock service instance using `jasmine`'s `spyOn`.

Note that `spyOn` must be called on the `ArticlesService` instance
that provided by `TestBed`,
not on the `mockArticlesService` object we created.

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.1.ts %}

The tests are then performed on these instances:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.2.ts %}

Putting it all together, the test code reads:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.ts %}

These tests exercise the intended interfaces and contracts of the component,
in the ways that the component would be used in the application.

## Summing Up

In this article, you should have learned:
* Why most Angular 2+ Component testing tutorial are wrong
* What are the public interfaces of an Angular 2+ Component and how to test them
