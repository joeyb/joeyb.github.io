---
layout: post
title: "Tight Coupling: A Brownfield Project Nightmare"
tags: [architecture, best-practices]
---

As a consultant, I get the opportunity to work on projects that are at all points of the Greenfield/Brownfield spectrum. Some developers take pride in revamping an old project, fixing architectural/design issues, and cleaning up other people's messes. I am **not** one of those developers. When I unfortunately land on a Brownfield project, the largest pain points almost always boil down to one design problem: [Tight Coupling][1].

In short, tight coupling results when lots of unrelated pieces of code directly depend on each other and changes to any one object would result in a large number of other changes throughout the project. In an object oriented project there will always be some coupling because objects need to communicate. One of the main goals when architecting a project should be to use whatever idioms/patterns your language of choice provides to minimize coupling. Tight coupling leads to brittle code that may be quick to initally develop, but is a complete nightmare to extend and maintain.

What amazes me is the number of "senior" developers who still write code with major coupling issues. I think the problem stems from the fact that many developers are shortsighted, especially in the consulting world. I hear a lot of programmers talk about optimizing code for performance, or learning new techniques/libraries that speed up initial development, but what should be most important is writing code that is highly adaptable and will be easy to change in the future. If I've learned anything from consulting, it's that no spec is concrete and what is true today will almost certainly differ from what is true tomorrow.

So how do you avoid coupling issues? Learn, understand, and practice the [SOLID principles][2]. I can't emphasize enough how important I think applying SOLID is to any software project. [There][3] is a [plethora][4] of [great][5] [articles][6] on SOLID and I implore every developer to at least be familiar with each of the principles.

Most projects are not finite in scope. Learning and implementing strategies that help apply the SOLID principles (i.e. programming to an interface, duck typing, DI/IoC, etc.) may not show immediate results, but when you go to change that code months/years down the road it will be much easier if you have avoided tight coupling. Your project will be less brittle and changes to any given object will have a much smaller impact on surrounding code.

[1]: http://en.wikipedia.org/wiki/Coupling_(computer_programming)
[2]: http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[3]: http://lostechies.com/derickbailey/2009/02/11/solid-development-principles-in-motivational-pictures/
[4]: http://trycatchfail.com/blog/post/SOLID-Things-Every-Senior-NET-Developer-Should-Know-Part-2.aspx
[5]: http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod
[6]: http://confreaks.com/videos/240-goruco2009-solid-object-oriented-design