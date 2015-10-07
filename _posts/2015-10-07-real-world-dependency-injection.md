# Real World Dependency Injection

Martin Fowler provides a [good overview of Dependency Injection and Inversion of Control](http://martinfowler.com/articles/injection.html),
so I'll skip the overview and assume that you are familiar with both concepts.

Most devs that I know who are familiar with DI tend to have strong feelings about it. Many of those feelings are negative because they've
seen how easy it is to abuse DI. This is definitely a great power, great responsibility scenario, and using DI successfully requires a
thorough understanding of the objects that you intend to inject. My goal here is to provide some guidelines for properly harnessing that
power and avoiding common pitfalls.

I'll use Java in my examples, but the same principals should apply to any language or DI framework.

## Prefer Constructor Injection Wherever Possible

Most DI frameworks provide a variety of methods for injecting objects (constructors, fields, methods, etc.). Constructor injection should be
preferred because it leads to code that is less tightly coupled to your DI framework, and therefore easier to test in isolation and re-use
outside the scope of your DI framework. For example, let's pretend we have a hypothetical `WidgetManager` interface:

```java
public interface WidgetManager {
    
    void startWidget(Widget widget);
}
```

And 2 different implementations of that interface, one that uses constructor injection and one that uses field injection:

```java
public class ConstructorInjectionWidgetManager implements WidgetManager {
    
    private final ThingService thingService;

    @Inject
    public ConstructorInjectionWidgetManager(ThingService thingService) {
        this.thingService = thingService;
    }

    public void startWidget(Widget widget) {
        // Use thingService to start the widget ...
    }
}
```

```java
public class FieldInjectionWidgetManager implements WidgetManager {
    
    @Inject
    ThingService thingService;

    public void startWidget(Widget widget) {
        // Use thingService to start the widget ...
    }
}
```

At first glance, the field-based implementation definitely looks simpler. The issue is that it's not obvious from the public interface of
that class what is required for it to function correctly. If you want to use it outside of the DI context (e.g. in a unit test), then you
*have* to read through the internals of the class to know that you'll need to set that non-public `thingService` field.

The constructor-based approach uses the type system to *force* users of that class to provide any needed dependencies. The field-based
approach can lead to subtle bugs that only show up at runtime if a dev erroneously manually constructs an instance and neglects to set a
required dependency. With the constructor-based approach, the compiler would prevent that type of error from ever happening.

## Avoid Depending On Lifetimes/Scopes

Most DI frameworks provide some way to specify a dependency instance's lifetime. The main examples are per-lookup (new instance on every
injection), singleton (same, single instance for every injection), and per-request (for web apps, same instance for every injection over
the course of a single web request).

Injecting objects that function differently based on their scope can lead to runtime issues that are difficult to diagnose and may require
major refactoring to fix. For example, let's use JPA's `EntityManager`. An `EntityManager` basically manages a single "unit of work" in
the database and is used to fetch entities, track their changes, and commit those changes back to the database. In order to commit an
entity instance to the database, it must be "tracked" by that instance of the `EntityManager`. Let's pretend we have two different
services:

```java
public class FetcherService {
    
    private final EntityManager entityManager;

    @Inject
    public FetcherService(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    public Widget getWidgetById(int id) {
        // Use the entityManager to fetch and return the requested widget
    }
}
```

```java
public class UpdaterService {
    
    private final EntityManager entityManager;

    @Inject
    public UpdaterService(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    public void updateWidgetName(Widget widget, String name) {
        // Use the entityManager update the widget name and save it to the database
    }
}
```

If the `EntityManager` DI binding is scoped as something like singleton (which is a terrible idea since it keeps a ton of state) or
per-request then you can fetch the widget in one service, pass it to the next service and update it without issue. On the other hand, if
the `EntityManager` DI binding is scoped as per-lookup (new instance on every injection) then things get more complicated. Each service
would therefore have a different instance of `EntityManager`, which means the `UpdaterService` would need to know that it has to attach
that instance of the widget to its context in order to start tracking it before it can update it. With `EntityManager` you also run into
a similar class of issues with transactions since each instance supports 1 (and only 1) concurrent transaction.

With classes like `EntityManager` that are so dependent on state and lifetime, you're probably better off manually managing their
lifetime in order to be as explicit as possible about where instances are being shared and where they are not.

In my experience the best uses for scopes are as an optimization for objects that are time consuming to construct, and for setting hard
bounds on the lifetime of a cache.
