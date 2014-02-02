---
published: false
layout: post
title: "Re-thinking Application Architecture: Designing to Promote Failure in Isolation"
tags: 
  - architecture
---

The majority of applications that I have worked on have been structured around the tried-and-true [N-Tier architecture][1] model. It is an intuitive pattern for separating concerns and most libraries/tools (especially in the .NET world) are built with that pattern in mind. For example, Entity Framework is generally used with one central `DbContext` inside a shared Data project.

The main issue with this pattern is that adding a new module to your application will generally require changes to core pieces of the shared architecture. In an ideal world you would trust any member of your team to make those types of changes, but in practice most teams have a wide range of skill levels and less experienced developers run a high risk of adding low quality code in these core shared areas.

That low quality code should of course be caught and fixed in your code review process, but code reviews rarely seem to completely fix sloppy code. Iteratively fixing low quality code through code review processes feels like you are just trying to [polish a turd][2]. What I really want is to let developers fail in isolation whenever possible and limit potential for that code affecting the rest of the system. It is inevitable that code quality will vary and it would be ideal to quarantine each module so that it can be refactored/rewritten with as little impact to the rest of the project as possible.

#### Re-orienting the Architecture: Moving from _Horizonal_ to _Vertical_ slices

In an attempt to solve this issue, I am working on designing a project architecture that moves away from the N-Tier model for separating projects by "category" (i.e. Data, Services, Web, etc.) towards separating them by "module" (i.e. Users, Reports, Products, etc.). The goal is keep all functionality related to that module grouped together and promote building modules that are as isolated as possible.

Obviously there will always be core shared code, but the goal here is to greatly reduce the need to modify that code. Anything that is shared between modules should have a much higher standard for quality and should probably be designed/developed by the more senior members of your team.

#### Re-architecting the Data Layer

The most difficult piece to handle in order to support the proposed re-organization is the Data Layer. Many ORM tools expect you to have a central place where you define the data entities being used in your application. For example, Entity Framework expects you do have a central `DbContext`. Each framework/language will have its own solution for how to handle this new architecture, but here are a couple ideas for doing it in .NET:

1. Continue to maintain a central `DbContext`, but move all of the data entities and any code for fetching/creating/manipulating them out into their respective modules. You still end up with that central pinch point, but you greatly reduce the amount of centralized code. The biggest issue here is that you will need to be very careful about how the projects and their references are setup or else you will end up in circular dependency hell.
2. Create a separate `DbContext` for each module. In most cases you'll still need to have a central `DbContext` for shared data, but this will reduce the number of changes required on the central one. The main negative here is that you will probably lose the ability to setup relations between entities that are in different contexts.
3. Move away from a full-fledged ORM to a simple data mapper like [Dapper][3]. This removes the need for a central `DbContext`-like object.

#### Re-architecting the Web Layer

As with the Data Layer, how you make changes to support the vertical slicing will depend on your framework/language, but in the .NET world there have been a bunch of changes recently that help facilitate the move to a more modular architecture.

If you are using ASP.NET MVC and/or WebAPI then the recent changes that support specifying routes as an action attribute (instead of centrally in the Global.asax), along with using reflection to find/register the controllers in each module should get you most of the way there. There will still be a central Web project, but it will basically just handle that controller discovery process.

For an MVC application, you will also need to figure out how you want to handle module-specific static content. I have not had to deal with that just yet so I don't have a detailed suggestion, but it should be possible to have that content built out into the main Web project's content directory.

#### Conclusion

This whole idea is still a work in progress. I am interested to get some more feedback and continue to tweak the architecture. The N-Tier approach of separating code by category makes sense from a "holistic" organization standpoint, but it does a poor job of organizing around how the code is actually developed.

My biggest concern about organizing around vertical slices is regarding long-term refactorability. For example, if you end up needing to make large-scale changes across the entire data layer then it may be easier if all of your data entities/services/etc. are centrally located in one monolithic project. With modern refactoring tools this may not be a real issue, but it is something that I am keeping an eye on.

[1]: http://en.wikipedia.org/wiki/Multitier_architecture
[2]: http://www.youtube.com/watch?v=yiJ9fy1qSFI
[3]: https://code.google.com/p/dapper-dot-net/