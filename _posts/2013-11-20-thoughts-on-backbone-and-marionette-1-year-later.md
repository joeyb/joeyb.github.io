---
layout: post
title: "Thoughts on Backbone and Marionette - 1 Year Later"
tags: [backbone, javascript, marionette]
---

I've been working on a large-scale Backbone/Marionette-based application for over a year now.
The entire process has been a great learning experience, especially regarding discovering where
the real pain points lie for architecting a large single page app.

We decided to use Backbone mainly due to its simplicity and straightforwardness. I had concerns
with the lack of JavaScript experience on the team and was worried that using a more complex
framework would pose too large of a learning curve. The biggest benefit to using Backbone for
a developer relatively new to JavaScript is its explicitness, but that also leads to a TON of
boilerplate code that is either minimized or eliminated in more advanced frameworks.

## Coordinating Data Loading

Our biggest pain point has been related to loading up all the data needed for a particular
page/module and being able to wait for those calls to fully resolve before considering the
page/module fully loaded. Neither Backbone nor Marionette have built-in support for anything
similar to Angular's route resolve methods. It makes sense that each view should be responsible
for loading its own data, but how does that view communicate back up to its parents when the
loading process has completed?

Assuming you're using a `Marionette.Controller` to coordinate each page/module, the simplest
approach would be to force all data loading to only occur within the controller, then let it
pass that down to any child views. This works, but I would much rather the actual view that
needs that data be in charge of fetching it where possible in order to reduce external
dependencies.

Another option would be to build support for a more advanced view life-cycle so that in addition
to the default events (i.e. render, close) there are more fine-grained steps for other phases
relevent to your app (i.e. load data, authorization). This could work, but it requires some
fairly significant changes on top of the Backbone/Marionette infrastructure to implement.

Our current compromise has been to use a page-specific instance of Backbone.Wreqr that is passed
down to every child view on a page so that we have a page-level event aggregator. We then use
ajax:start/ajax:stop events to notify a global listener when data is being loaded, increment a
counter whenever there is an ajax:start and decrement it whenever there is an ajax:stop. Whenever
that count is greater than 0 then we know there is still an active ajax request. It definitely
feels brittle and I'm not 100% happy with the implementation, but it works for now and it
involved less invasive changes than building out the full view life-cycle described above.

## Backbone.Model

The model is the one piece of Backbone that is really opinionated, and we have ended up spending
more time fighting those opinions than taking advantage of them.

Our app is very read-heavy. We have some endpoints that are non-RESTful for various reasons (mainly
related to reads which need unbounded inputs, so they have to be done as POSTs). We are able to
handle all of our use cases, but the only functionality we really use from `Backbone.Model` is
custom response parser. At this point I strongly prefer Angular's approach to have both a
lower-level `$http` service and a higher-level `$resource` service built on top of it with
enhancements for interacting RESTful services.

## Prototypal Inheritance Is Evil

Google's JavaScript style guide encourages developers to avoid prototypal inheritance wherever
possible. For developers coming from other languages (especially C#), this may seem counterintuitive.
I admit that when I first started doing heavy JavaScript development I had a tendency to lean towards
creating "reusable" base classes far too often. They seem great in theory, but inheritance gets
complicated very quickly in JavaScript. Deep class hierarchies have a tendency to include a huge
number of `BaseClass.prototype.methodName.apply(...)` calls littered throughout. Debugging that gets
complicated quickly.

The archtecture of both Backbone and Marionette tends to promote those deep class hierarchies.
Marionette's `CompositeView` has the following class lineage:

`Marionette.CompositeView` -> `Marionette.CollectionView` -> `Marionette.View` -> `Backbone.View`

Both frameworks are inconsistent about where/how methods are overridden so debugging what happens
where can be difficult. It can be far too easy to violate the Liskov Substitution Principle. In
JavaScript I've had much better luck prefering composition over inheritance. Composition is really
only possible when given the right extension points and there are a lot of places in both Backbone
and Marionette where those extension points aren't made available.

## Conclusion

I still stand behind my initial reasoning for deciding to use Backbone, but in hindsight it may have
been worth the increased learning curve for the team to use a framework like Angular that does a better
job of handling these pain points. In my experience with Angular, we probably could have accomplished
the same functionality in nearly half the lines of code because of all the boilerplate required by Backbone.
