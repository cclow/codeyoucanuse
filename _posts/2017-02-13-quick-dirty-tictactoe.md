---
layout: post
title: Quick and Dirty TicTacToe
---
A common criticism of Angular is that it is complicated compared with
ReactJS or Vue.js.
While this is not wrong, it is important to remember the context for this complexity.

ReactJS and Vue.js are view libraries,
and view libraries are just a part of the Angular framework.
Even when we compare view libraries, Angular's Components, Directives, and Pipes
are much more feature rich than ReactJS and Vue.js.
Furthermore, you'll find that much of Angular's complexity are
enhancements for modularity and scalability,
important features for team development, though not so much for 
simple, one man developed web apps.
You can easily ignore these enhancements and structures if you choose.

What if, we do just that?
Using just Angular's view library implementing a quick and dirty app, say
the TicTacToe game?
This should give us a better way to compare similar development efforts.

<iframe width="640" height="360" src="https://www.youtube.com/embed/dfIkIptSaC8" frameborder="0" allowfullscreen></iframe>

### Summing Up

Through this exercise, we demonstrated that if we use only
the view library of Angular, and choose not to adhere to Angular's style guides,
developing in Angular can be almost as "easy" as in other view libraries.

There are some unavoidable overheads such as the need to declare each component,
and there are unfamiliar syntax arising from Typescript, decorators, templates,
that might require some effort to learn.

A case could still be made that the initial learning curve can be lessen, 
and these efforts are worthwhile for the benefits provided by the much larger and
feature rich Angular framework.
 