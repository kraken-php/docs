# Service Container

- [Introduction](#introduction)
- [Binding](#binding)
    - [Binding Basics](#binding-basics)
    - [Sharing](#sharing)
    - [Removing Definitions](#removing-definitions)
- [Resolving](#resolving)
    - [Making](#making)
    - [Auto Wiring](#auto-wiring)

<a name="introduction"></a>
## Introduction

Kraken service container is a powerful tool for managing class dependencies and performing dependency injection. Dependency injection is a process of configuring your concrete objects via "injecting" its dependencies via the constructor or setter method. It allows to decouple components in your application in order to write clean and testable code. You can read more about dependency injection in [this document](http://www.phptherightway.com/#dependency_injection).

<a name="binding"></a>
## Binding

The most of your service container bindings will be registered within [service providers](/docs/{{version}}/service-providers), so the examples provided will focus on using the container in that context.

> {tip} There is no need to bind classes into the container if they do not depend on any interfaces or primitve values. The container is able to build these objects automatically resolving its dependencies using [auto wiring](#auto-wiring).

<a name="binding-basics"></a>
### Binding Basics

Almost all of your bindings can be done using `bind` method.

    $container->bind($reference, $mixed);

> {warning} In typical cases `bind` method should be enough for your needs. However, there are some cases - like invokable objects - on which `bind` method's behaviour might be undesirable. To avoid this kind of an issue, you might use specialized methods presented in the next part of this document. 

#### Binding Classes

You can bind classes or alias using `alias` method.

    $container->alias(ClassOrInterface::class, $otherClassOrAlias);

#### Binding Instances

When you need to bind concrete instance of class to container, you can do it via `instance` method.

    $container->instance(ClassOrInterface::class, $concrete);

#### Binding Factory Methods

To bind a factory method that will be invoked each time when object is made, use `factory` method.

    $container->factory(ClassOrInterface::class, function() {
        return new Class;
    });

#### Binding Default Params

To wire default params to classes use can use `wire` method.

    $container->wire(ClassOrInterface::class, [ $param ]);  

#### Binding Primitives

To create primitive you should always use `param` method.

    $container->param($paramName, $paramValue);

<a name="sharing"></a>
### Sharing

There might be cases in your application, that you want to declared definition that should only be resolved one time. Once a singleton binding is resolved, the same object instance will be served each time container calls for it.

You can mark container definitions as singletons using simple `share` method.

    $container->share(ClassToShare::class);

<a name="removing-definitions"></a>
### Removing Definitions

Unlike other implementations of service container pattern, Kraken allows you to remove previously bound definitions. To do this, simply use `remove` method.

    $container->remove(ClassOrInterface::class);

> {notice} Keep in mind, that you are able to remove only custom made definitions. After removing them, object still will be resolveable, but using not-configured default objects via reflection.

<a name="resolving"></a>
## Resolving

<a name="making"></a>
### Making

To create instance of definition registered in service container you need to call `make` method with class name you wish to resolve.

    $container->make(ClassOrInterface::class);

If you want to overwrite default params of class being made, you can specify new params.

    $container->make(ClassOrInterface::class, $params);

<a name="auto-wiring"></a>
### Auto Wiring

Kraken service container has the power to automatically resolve objects and their dependencies recursively by inspecting the type hints of your constructor arguments via reflections. This, however, does not work for objects that require primitive values or interfaces.

You can read more about service container in its own [section](/docs/{{version}}/api-container). The default providers shipped with the Kraken Framework can be found in [service providers section](/docs/{{version}}/service-providers).

