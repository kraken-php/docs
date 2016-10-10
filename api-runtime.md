# Runtime Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Changing The State](#changing-the-state)
    - [Registering Listeners](#registering-listeners)
    - [Getting Context](#getting-context)
    - [Getting State](#getting-state)
    - [Triggering The Supervisor](#triggering-the-supervisor)
    - [Creating & Destroying Children](#creating-and-destroying-children)
    - [Starting & Stopping Children](#starting-and-stopping-children)
    - [Sending Messages](#sending-messages)
    - [Sending Requests](#sending-requests)
    - [Invoking Remote Commands](#invoking-remote-commands)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [RuntimeContainer](#runtime-container)
    - [RuntimeContainerInterface](#runtime-container-interface)
    - [RuntimeModel](#runtime-model)
    - [RuntimeModelInterface](#runtime-model-interface)
    - [RuntimeContextInterface](#runtime-context-interface)
    - [RuntimeCommand](#runtime-command)
    - [RuntimeManager](#runtime-manager)
    - [RuntimeManagerInterface](#runtime-manager-interface)
- [Important Directories](#important-directories)
    - [Command](#command)
    - [Container](#container)
    - [Supervision](#supervision)

<a name="introduction"></a>
## Introduction

Runtime is component that provides container-based abstraction for Threads and Processes and means of managing and supervising children containers from its ancestor level.

<a name="introduction"></a>
## Features

Runtime features:

<div class="dot-list" markdown="1">
- Container-based (actor-based) abstraction for Threads and Processes
- Separation between standard and emergent flow of business logic inside single container
- Command-based controls to pass orders between containers
- Built-in Process local and remote managers
- Built-in Thread local and remote managers
- Built-in Runtime managers abstracting managment of processes and threads
- Supervision mechanisms with separation of local and remote errors
- Supervision problem solvers
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Runtime

Runtime is a common name for both process or thread created by Kraken. It is an abstractions that allows operating on them using single interface.

### Container

Container is an instance of any given runtime that contains both runtime's logic and all of its services. Container is Kraken's implementation of an *actor*.

You can read more about runtimes in [runtimes article](/docs/{{version}}/runtimes).

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="changing-the-state"></a>
### Changing The State

Changing the state of runtime might be done via `start` and `stop` methods.

    // this will emit application-wide "start" event
    $container->start(); 
    
    // this will emit application-wide "stop" event
    $container->stop();
 
> {warning} There exists also `create` and `destroy` methods, however you must never call them directly, as using them directly might danger your application integrity.

> {notice} The `start` and `stop` methods are designed mainly for your application logic to respond to container downtimes and uptimes. Kraken internal services does not react to them, as they have to work anyway.

<a name="registering-listeners"></a>
### Registering Listeners

There are many of events thrown by container during its lifecycle. You can view them in `Kraken\Runtime\RuntimeContainerInterface`. The most important two are `start` and `stop`.

    $container->onStart(function() {
        // do something on container start
    });
    
    $container->onStop(function() {
        // do something on container stop
    });

<a name="getting-context"></a>
### Getting Context

The container interface extends `Kraken\Runtime\RuntimeContextInterface` so you are able to use methods such as `getType`, `getParent`, `getAlias`, `getName` and `getArgs` for getting all information about current container context.

Getting alias example:

    $alias = $runtime->getAlias();

<a name="getting-state"></a>
### Getting State

The container's state might be fetched using following syntax:

    $state = $runtime->getState();

<a name="triggering-the-supervisor"></a>
### Triggering The Supervisor

Triggering supervisor will force your container to stop application logic and enter `failed` state. You can read more about this in [supervision article](/docs/{{version}}/supervision). To trigger the supervision use `fail` method.

    $runtime->fail($exception, $params);

<a name="creating-and-destroying-children"></a>
### Creating & Destroying Children

Creating and destroying the children might be done using create and destroy prefixed method of `Kraken\Runtime\RuntimeManagerInterface`. All these methods implements `Kraken\Promise\PromiseInterface`.

#### Creating Process

    $manager = $runtime->getManager();
    $manager
        ->createProcess($alias, $name, $flags)
        ->done(function() {
            echo "Process has been created!\n";
        });

#### Creating Thread

    $manager = $runtime->getManager();
    $manager
        ->createThread($alias, $name, $flags)
        ->done(function() {
            echo "Thread has been created!\n";
        });

#### Destroying Process

    $manager = $runtime->getManager();
    $manager
        ->destroyProcess($alias, $name, $flags)
        ->done(function() {
            echo "Process has been destroyed!\n";
        });

#### Destroying Thread

    $manager = $runtime->getManager();
    $manager
        ->destroyThread($alias, $name, $flags)
        ->done(function() {
            echo "Thread has been destroyed!\n";
        });

> {tip} Some of the process and thread related methods can be abstracted via runtime related methods that works with both of them. The only abstraction which is not provided by runtime methods is creation of containers, as you have to specify exactly what kind of runtime you want to invoke.

<a name="starting-and-stopping-children"></a>
### Starting & Stopping Children

Starting and stopping the children might be done in similar way as creating and destroying them. The difference is that you have to use start and stop prefixed methods:

#### Starting Container

    $manager = $runtime->getManager();
    $manager
        ->startRuntime($alias, $name, $flags)
        ->done(function() {
            echo "Container has been started!\n";
        });

#### Stopping Container

    $manager = $runtime->getManager();
    $manager
        ->stopRuntime($alias, $name, $flags)
        ->done(function() {
            echo "Container has been stopped!\n";
        });

> {tip} Some of the process and thread related methods can be abstracted via runtime related methods that works with both of them. The only abstraction which is not provided by runtime methods is creation of containers, as you have to specify exactly what kind of runtime you want to invoke.

<a name="sending-messages"></a>
### Sending Messages

Sending messages to runtime container's children might be done via:

    $manager = $runtime->getManager();
    $manager->sendMessage($alias, $message, $flags);

<a name="sending-requests"></a>
### Sending Requests

Sending requests to runtime container's children might be done via:

    $manager = $runtime->getManager();
    $manager
        ->sendRequest($alias, $message, $params)
        ->done(function($result) {
            printf("Received response = \"%s\"\n", $result);
        });

<a name="invoking-remote-commands"></a>
### Invoking Remote Commands

Remote commands might be invoked on runtime container's children using following syntax:

    $manager = $runtime->getManager();
    $manager
        ->sendCommand($alias, $commandName, $commandParams)
        ->done(function($result) {
            printf("Received response = \"%s\"\n", $result);
        });

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="runtime-container"></a>
### RuntimeContainer

    class RuntimeContainer extends EventEmitter implements RuntimeContainerInterface

RuntimeContainer is a concrete class that contains all internal data of container including the context and the core, it also works as container controller.

<a name="runtime-container-interface"></a>
### RuntimeContainerInterface

    interface RuntimeContainerInterface extends RuntimeContextInterface, CoreGetterAwareInterface, EventEmitterInterface, LoopGetterAwareInterface

<a name="runtime-model"></a>
### RuntimeModel

    class RuntimeModel implements RuntimeModelInterface

RuntimeModel is internal implementation working behind RuntimeContainer class.

<a name="runtime-model-interface"></a>
### RuntimeModelInterface

    interface RuntimeModelInterface extends RuntimeContextInterface, RuntimeManagerAwareInterface, SupervisorAwareInterface, CoreAwareInterface, EventEmitterAwareInterface, LoopExtendedAwareInterface

<a name="runtime-context-interface"></a>
### RuntimeContextInterface

    interface RuntimeContextInterface

RuntimeContextInterface is an interface providing set o methods for accessing container's context information.

<a name="runtime-command"></a>
### RuntimeCommand

    class RuntimeCommand

RuntimeCommand is a class that have to be used when invoking remote commands between containers using their channels.

<a name="runtime-manager"></a>
### RuntimeManager

    class RuntimeManager implements RuntimeManagerInterface

RuntimeManager is a class that allows managing container's children.

<a name="runtime-manager-interface"></a>
### RuntimeManagerInterface

    interface RuntimeManagerInterface extends ProcessManagerInterface, ThreadManagerInterface

<a name="important-directories"></a>
## Important Directories

This section contains list of most important directories existing inside of component. It **does not include all** directories.

<a name="command"></a>
### Command

Command folder contains pre-defined container-related commands.

<a name="container"></a>
### Container

Container folder contains internal logic working behind runtimes abstraction and useful container-related classes such as controllers and managers.

<a name="supervision"></a>
### Supervision

Supervision folder contains pre-defined problem solvers.
