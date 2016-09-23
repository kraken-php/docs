# Events & Events-Loop

- [Introduction](#introduction)
- [Event-driven Architecture](#event-driven-architecture)
    - [Definition](#definition)
    - [Emitting & Listening](#emitting-and-listening)
    - [Event-Loop](#event-loop)
    - [Queuing](#queuing)
    - [Using Timers](#using-timers)
    - [Using Streams](#using-streams)
    - [Catching Throwables](#catching-throwables)
- [Advenced Patterns](#advanced-patterns)
    - [Registering Temporary Listeners](#registering-temporary-listeners)
    - [Registering Delayed Listeners](#registering-delayed-listeners)
    - [Changing Loop Model](#changing-loop-model)
    - [Using Multiple Loops](#using-multiple-loops)
    - [Using React Bridge](#using-react-bridge)

<a name="introduction"></a>
## Introduction

The most important feature of Kraken Framework architecture is it being **event-driven**, which allows you to write fully asynchronous and scallable applications. You might not be familiar with this architectural design by its name, but it is almost sure that you encountered it already in other programming langauges, like in JavaScript for example.

This article will focus on describing how event-driven architecture is implemented in Kraken and presenting offered possibilites by framework.

<a name="event-driven-architecture"></a>
## Event-driven Architecture

<a name="definition"></a>
### Definition

Event-driven architecture is a software architecture pattern promoting applications and systems which transmit events among loosely coupled components and services. This kind of systems usually consists of event emitters and event listeners and are frequently used for implementing [asynchronous](/docs/{{version}}/asynchronity) components.

#### JavaScript Example

This definition is hard to understand at first, so real-world example of using events could help. Let's talk about JavaScript for a while. In JavaScript events are used to signal change of state of many things, but most frequently, DOM objects. If you have created at least one HTML website you probably have used events not knowingly. Try to remember how function can be registered to be called when any HTML element is clicked. It usually goes like this:

    object.onclick = function() {
        // object has been clicked, do something!
    };

In above example, registered function might be considered an **event listener** that waits for the `click` **event**, that might be fired by object being **event emitter**. It is exemplary use of events. JavaScript itself is event-driven as it this kind of tools by itself.

#### PHP Example

PHP does not provide any tools for using events out of the box. They have to be made, and this is exactly what Kraken does. It provides your application a set of tools that allows you using event-driven architecture in PHP. The presented article will focus mainly on core tools, centered arount the idea of [events](#events) and [events-loop](#events-loop).

Previous example could be written in PHP using Kraken Event tools in the following way:

    $object->on('click', function() {
        // object has been clicked, do something!
    });

`$object` is an instance of any `EventEmitter` which provides `on` method that automatically creates `EventListener` with handler being an anonymous function.

<a name="emitting-and-listening"></a>
### Emitting & Listening

To utilize events in your application you have to use both **event emitters** and **event listeners**. Event emitter is an object which purpose is to emit event while event listener is an object that calls proper handler in response. 

There are two types of emitters included with Kraken Framework, synchronous `Kraken\Event\BaseEventEmitter` and asynchronous `Kraken\Event\AsyncEventEmitter`. Both of them can be constructed using an abstraction of `Kraken\Event\EventEmitter`.

To emit an event use `emit` method.

    $emitter = new EventEmitter();
    $emitter->emit('myEvent');

Additionaly, you can parametrize the event by passing an array of values in second argument, like:

    $emitter->emit('myEvent', [ $firstParam, $secondParam ]);

The `emit` function will iterate through the collection of registered listeners to `myEvent` event, and call all of them in the order they were registerd, one after another. If not one listener was registered during emitting of event, nothing will happen.

To create event listener `on` method should be used:

    $emitter->on('myEvent', function() {
        // do something with event
    });

To catch params passed with event, you can parametrize the function handler itself:

    $emitter->on('myEvent', function($param1, $param2) {
        // do something with event
    });

Aside from `on` method there are a few others useful methods for creating listeners that you can read about in [advanced patterns section](#advanced-patterns) or [event component article](/docs/{{version}}/api-event).

> {tip} Each one of your application components should extend or implement its own event emitter. In no case, you should event consider of using centralized event emitter anti-pattern.

<a name="event-loop"></a>
### Event-Loop

In event-driven application keeping track and ordering the events might become quite troublesome task, especially when events transfering start to become mainly non-blocking. To aid you with this problem, Kraken provides you its custom event-loop implementation, which acts as a central location for registering and ordering interests.

The event loop schedules listeners, runs timers, handles signals, and polls streams for pending reads and available writes. There are several event loop backgrounds available depending on your application environment. The `Kraken\Loop\Model\SelectLoop` is the basic one using PHP `stream_select` function. It will work on any PHP installation, but is not as performant as other available models. You can change this model to another via `loop.model` configuration option. All event loops implement `Kraken\Loop\LoopInterface` and provide the same features. 

It is important to know, that only one loops can be active at time, and internal instance is created automatically by Kraken. You are able to refer to this instance using `Loop` service or resolving `Kraken\Loop\LoopInterface` object via service container. You should never have to create new loop by scratch.

<a name="queuing"></a>
### Queuing

Queuing functions to event loop allows your application to become more responsive, as the loop will take control of all scheduling and executing. To queue any callbacks use `onTick` method.

    $loop->onTick(function() {
        // this will be executed as soon as possible!
    });

For more control about how callbacks are qeued you can also consider using `onBeforeTick` or `onAfterTick` methods.

<a name="using-timers"></a>
### Using Timers

Timer is special kind of function which execution is delayed by given time interval. `Kraken\Loop` component manages timers automatically and does its best to ensure all timers all executed properly.

To create timers you can use `addTimer` method.

    $loop->addTimer(0.5, function() {
        // this will be executed after half of a second
    });

To create periodic timers, you `addPeriodicTimer` instead.

    $loop->addPeriodicTimer(1.0, function() {
        // this will be executed every each one second
    });

<a name="using-streams"></a>
### Using Streams

Kraken loop also allows you to create listeners for periodic and asynchronous read & write using streams.

To read data from streams use:

    $loop->addReadStream($resource, function($resource) {
        // any reading operation should be done in this section
    });

To write data to streams use:

    $loop->addWriteStream($resource, function($resource) {
        // any writing operation should be done in this section
    });

<a name="catching-throwables"></a>
### Catching Throwables

All unhandled throwables thrown inside of event will make your application to enter `failure` state, and try to solve the problem using local superivision system, which will then decide what should be done.

<a name="advanced-patterns"></a>
## Advanced Patterns

<a name="registering-temporary-listeners"></a>
### Registering Temporary Listeners

To register temporary listeners, that will unregister itself after one or specified number of executions use `once` or `times` method.

    $emitter->once('myEvent', function() {
        // this will be executed only once!
    });

    $emitter->times('myEvent', 3, function() {
        // this will be executed at most three times!
    });

<a name="registering-delayed-listeners"></a>
### Registering Delayed Listeners

To register delayed listeners, that will start to be executed after specified number of events have already been fired, use `delay` `delayOnce` or `delayTimes` method.

    $emitter->delayOnce('delayed', 3, function($i) {
        printf("%s\n", $i);
    });
    
    for ($i=0; $i<5; $i++)
    {
        $emitter->emit('delayed', [ $i ]);
    }

Preceding example would emit five `delayed` events, but listener would be executed only once, on third event, printing `3` number.

<a name="changing-loop-model"></a>
### Changing Loop Model

All of the available loop models can be found in `Kraken\Loop\Model` namespace. The default used loop is `Kraken\Loop\Model\SelectLoop` which does not need any PHP extension to work. To change the model used, open Kraken configuration file and change the `loop.model` option, then restart your project.

<a name="using-multiple-loops"></a>
### Using Multiple Loops

Kraken allows you to used multiple loops by providing `import` and `export` methods. As the name suggests these methods allow you to save your loop state and export it to or import from extenal storage. Thanks to that feature, multiple loops and switching between them can be done. 

In default configuration, Kraken Framework uses two loops. The frist one is used under normal execution flow, and second one for `failure` state. This ensures, that your application will be able to respond to failures immediataly as they happen allowing you to solve problems using loop queue safely.

<a name="using-react-bridge"></a>
### Using React Bridge

To be able to use React-compatible libraries, you need to provide them instance of loop that implements `React\EventLoop\LoopInterface`. This can be done by wrapping Kraken loop inside of `Kraken\Loop\Bridge\React\ReactLoop`, like:

    $reactLoop = new \Kraken\Bridge\React\ReactLoop($krakenLoop);
