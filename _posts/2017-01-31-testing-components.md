---
layout: post
title: Testing Angular 2+ Components The Right Way
---
Most Angular 2+ testing tutorials (including the official tutorial on angular.io) recommends
the wrong way to test Components.

Consider the following component:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e sut.component.ts %}

and its test code:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e sut.component.spec.ts %}

The test code reaches into the component to modify the value of `title`.
This is problematic in a few ways.

#### Problem 1: The test is fragile, and can break even when the implementation is correct.

This test makes assumptions into the implementation of SutComponent,
i.e. it assumes that it has a field called `title`
whose value should be displayed in the `h1` element.

If for some reason we need the field to be `someOtherName`,
but the property name remains as `title`:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e prob-1.ts %}

This implementation would still work correctly
but the test would now fail.

#### Problem 2: The test is incomplete, it can pass even if implementation is broken.

`SutComponent` was intended to have a property `title` that a parent could set.
The test case does not test this, and instead tests for a field named `title`.

The component would not work as intended, but
the following would still pass the test as specified above.

{% gist cclow/44ac8952fcc2416197f62ae963c9797e prob-2.ts %}

### The Right Way

To test Angular 2+ Components correctly,
we should consider the component as a black box during testing.

In other words, our tests must be written without knowledge or assumption
of the component's internal implementation details.
Tests must be written to test a component's public interfaces and contracts and *only* 
its public interfaces and contracts.

A well-encapsulated component can have the following types of public interfaces:
* Property binding, as illustrated above
* Event bindings, through which a component can emit an event to its parent
* UI rendering and interactions
* Interactions with models, business logic, and external systems through 
dependency-injected services

Now consider the following component that contains
all these types of public interfaces:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.ts %}

All the fields and methods are `private`
to enforce this strict encapsulation.

To test `ArticleComponent`, we start with a mock parent component that
binds to the properties and events of `ArticleComponent`, 
and a mock instance `ArticlesService`:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.0.ts %}

We instantiate the `TestParent` component instead of `ArticleComponent`, and then
extract the test fixture from the `TestParent` fixture.

We use dependency injection to inject `mockArticlesService` as a provider
for `ArticlesService`, and then use `jasmine`'s `spyOn` to spy on this
mock service instance.
Note that `spyOn` must not be called on `mockArticlesService`, but we must
`spyOn` the injected instance which we get from `TestBed`.

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.1.ts %}

The tests are then performed on these instances:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.2.ts %}

Putting it all together, the test code reads:

{% gist cclow/44ac8952fcc2416197f62ae963c9797e article.component.spec.ts %}