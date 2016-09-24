# Runtimes

- [Introduction](#introduction)
- [Container-based System](#container-based-system)
    - [Container Definition](#container-definition)
    - [Container Reference](#container-reference)
    - [Hierarchical Structure](#hierarchical-structure)
    - [State](#state)
    - [Behaviour](#behaviour)
    - [Messaging](#messaging)
    - [Supervision](#supervision)
    - [Child Containers](#child-containers)
- [Working With Runtimes](#working-with-runtimes)
    - [Writing Runtimes](#writing-runtimes)
    - [Resolving & Referencing](#resolving-and-referencing)
    - [Getting Context](#getting-context)
    - [Getting State](#getting-state)
    - [Operating On Children](#operating-on-children)
    - [Using Console](#using-console)

<a name="introduction"></a>
## Introduction

Runtimes are common name for both processes and threads created by Kraken. They are abstractions that allows operating on them using single interface. This article describes how runtimes are implemented and how they can be used in your application along with tips and best-practices

<a name="container-based system"><a>
## Container-based System

<a name="container-definition"></a>
### Container Definition

Container is an instance of any given runtime that *contains* both runtime's logic and all of its services. Container is Kraken's implementation of an **actor**, therefore Kraken Framework as a whole can be defined as **actor-based system**, but because of chosen naming convention all of documentation pages referes to it as **container-based system**. Keep in mind, that both of these definitions are in fact different names for the same concept.

The most important parts that composes typical container are [state](#state), [behaviour](#behaviour), [messaging mechanisms](#messaging), [supervision mechanisms](#supervision) and [child containers](#child-containers). All of them are encapsulated behind the [container reference](#container-reference). Noteworthy aspect about containers is that they have an explicit lifecycle and are not automatically destroyed when no longer referenced. It is your responsibility to make sure that each container will eventually be destroyed. This gives you control over how resources are released when any container is destroyed.

<a name="container-reference"></a>
### Container Reference

Each container is an isolated part of Kraken architecture, closed inside of its own runtime. It is impossible to reference objects or call functions directly on separate container. To be able to communicate between containers one can use container reference. Container reference is an unique alias that you passed during creation of container alongside its name. This alias can be treated as container identifier.

<a name="hierarchical-structure"></a>
### Hierarchical Structure

Containers form hierarchies. One container, which covers some functional need in the application might want to split up its task into smaller, more manageable pieces. For this purpose it starts child containers which it supervises. The details of supervision and the underlying concepts can be found in separate, [supervision article](/docs/{{version}}/supervision).

<a name="state"></a>
### State

Container might be in one of five states: `created`, `destroyed`, `started`, `stopped` and `failed`.

#### The `created` State

The `created` state is being granted to container after it has finished all of its constructing, booting and configuring. `created` state might change to either `started` or `destroyed`.

#### The `destroyed` State

The `destroyed` state is the first and the last state of container lifecycle. It is granted to container eiter shortly after creation of runtime or just before destructing its runtime. `destroyed` state might change only to `created` state.

#### The `started` State

The `started` state is the state that occurrs when not only Kraken but also your application logic has started. `started` state might change to either `stopped` or `failed`.

#### The `stopped` State

The `stopped` state is the state in which your application logic should close all publicly opened ports and halt all procesing until its started again. It might be compared to maintenance mode in other popular frameworks. The `stopped` state might change once again to `started` state or to `destroyed` one.

#### The `failed` State

The `failed` state is special kind of state that is invoked during `started` state when unhandled exception is thrown. It stops all execution forcibly, changes loop flow to emergency and notifies local supervision system about problem. The `failed` state might change to `started` or `stopped` state.

<a name="behaviour"></a>
### Behaviour

Behaviour of a container is the custom application logic that you provided. It combines set of commands, services and cyclic operations. The bahviour is set on the runtime creation, but may change over time as your application architecture reacts to your application needs or container itself changes its state. The initial behavior defined during construction of the container is special in the sense that a restart of the container will reset its behavior to this initial one.

<a name="messaging"></a>
### Messaging

Interaction between isolated containers should follow messaging-pattern, meaning that the only way of interacting between containers should be done via sending and receiving messages. Kraken provides its containers with special channels that performs inter-container communication out of the box. Each container has exactly one channel to which all senders enqueue their messages. Enqueuing happens in the time-order of send operations, which means that messages sent from different containers may not have a defined order at runtime due to the apparent randomness of distributing containers across threads and processes. Sending multiple messages to the same target from the same container, on the other hand, will enqueue them in the same order.

> {notice} It is recommended to read more about how messaging is done in documentation's separate [messaging article](/docs/{{version}}/messaging).

<a name="supervision"></a>
### Supervision

Kraken handles all faults transparently, applying one of the strategies described in [supervision and monitoring](/docs/{{version}}/supervision) for each incoming failure. There are two layers of supervision system. The first, one called *local supervisor* allows each of your containers to fix its own problems. If local supervision system is not able to solve the problem that occurred, it is able to delegate the issue to *remote supervisor*, which then will decide what two do. Local supervisor is defined in the container's own memory while remote supervisor is a supervisor existing in the parent container.

> {notice} It is recommended to read more about how Kraken implements supervision system in its's separate [supervision article](/docs/{{version}}/supervision).

<a name="child-containers"></a>
### Child Containers

Each container is able to create additional containers for delegating sub-tasks, and if it does, it automatically becomes their supervisor. The list of children is maintained within the container's runtimes manager. Modifications to the list are done by creating and destroying children, which happen behind the scenes in an asynchronous way.

<a name="working-with-runtimes"></a>
## Working With Runtimes

This section contains basic patterns for using Kraken containers.

<a name="writing-runtimes"></a>
### Writing Runtimes

To write a runtime, you have to create a class in `Process` or `Thread` application namespace. In default configuration the paths are `App\Process` and `App\Thread`. Created class have to extend `Kraken\Runtime\RuntimeContainer` and implement `Kraken\Runtime\RuntimeContainerInterface`. After doing that Kraken Framework will have an access for creating this kind of runtimes, but won't construct them without your command to do so.

Your runtime may look like this:

    class TestContainer extends RuntimeContainer implements RuntimeContainerInterface
    {
        protected function config(CoreInterface $core)
        {
            // this method should return additional configurations for this runtime if needed
            // in most cases you won't be using this at all since the main place of configuring
            // your application should take place in 'data\config' directory
            return [];
        }

        protected function boot(CoreInterface $core)
        {
            // this method should contain additional boot logic for this runtime if needed,
            // in most cases you won't be using this at all since the main place of booting
            // your application should take place in 'data\bootstrap' directory
            return $this;
        }
    
        protected function construct(CoreInterface $core)
        {
            // this is the method that will be executed after creating, configuring and booting
            // is completed, it is the most important part of runtime and should contain your
            // application logic
            return $this;
        }
    }

> {warning} Remember that the default pattern for runtimes is "App\%type%\%name%\%name%Container", if you want to create a thread-based runtime under the name of Test its namespace should be "App\Thread\Test\TestContainer". This pattern can be adjusted in `kraken.process` and `kraken.thread` files.

To run this runtime you should either code it into default `Main` runtime that is provided or dynamically using `kraken process:create Main T1 Test` command. T1 here is an aliased for referencing container which is going to be created from outside.

> {tip} If you have problem with creating containers, check if channel ports are configured properly.

<a name="resolving-and-referencing"></a>
### Resolving & Referencing

To access current Kraken container, you can resolve `Runtime` service or `Kraken\Runtime\RuntimeContainerInterface` interface using service container.

    // this
    $runtime = $container->make('Runtime');
    
    // or this
    $runtime = $container->make(RuntimeContainerInterface::class);

<a name="getting-context"></a>
### Getting Context

The container interface extends `Kraken\Runtime\RuntimeContextInterface` so you are able to use methods such as `getType`, `getParent`, `getAlias`, `getName` and `getArgs` for getting all information about current container context.

    $alias = $runtime->getAlias();

<a name="getting-state"></a>
### Getting State

The container's state might be get using following syntax:

    $state = $runtime->getState();

<a name="operating-on-children"></a>
### Operating On Children

All operations on child containers, including creating, destroying, starting and stopping can be done via `Kraken\Runtime\RuntimeManagerInterface`. It can be resolved from within current container via:

    $manager = $runtime->getManager();

For example, creating a child container might look like this:

    $manager = $runtime->getManager();
    $manager
        ->createProcess($containerAlias, $containerName)
        ->done(function($result) {
           echo $result . PHP_EOL;
        });

> {notice} It is safe to contain `createProcess` calls on runtime's construction, because Kraken checks for runtime existence before trying to create it. If it exists, the function will return success, without creating duplicate.

To execute command on one of child containers, use:

    $manager = $runtime->getManager();
    $manager
        ->sendCommand($containerAlias, $commandName, $commandParams)
        ->done(function($result) {
            // $result is command returned value
        });

> {notice} RuntimeManager provides also `sendMessage` and `sendRequest` for using non-command related communication.

<a name="using-console"></a>
### Using Console

Kraken allows you to adapt your application on the fly, without a need to hardcode hierarchy into your application. For, example adding B process-based container implementing `App\Process\MyProc\MyProcContainer` class as a child of A container, can be done via console:

    $> kraken process:create A B MyProc

Kraken console offers a set of useful commands for adapting and managing your application. You can read more about them in [console article](/docs/{{version}}/console-client).
