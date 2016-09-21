# Event Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Creating Base Emitters](#creating-base-emitters)
    - [Creating Asynchronous Emitters](#creating-asynchronous-emitters)
    - [Emitting Events](#emitting-events)
    - [Attaching Listeners](#attaching-listeners)
    - [Attaching Single-Execution Listeners](#attaching-single-execution-listeners)
    - [Attaching Multiple-Execution Listeners](#attaching-multiple-execution-listeners)
    - [Delaying Listeners](#delaying-listeners)
    - [Cancelling Listeners](#cancelling-listeners)
    - [Forwarding & Discarding Events](#forwarding-and-discarding-events)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [AsyncEventEmitter](#async-event-emitter)
    - [BaseEventEmitter](#base-event-emitter)
    - [EventEmitter](#event-emitter)
    - [EventEmitterInterface](#event-emitter-interface)
    - [EventHandler](#event-handler)

<a name="introduction"></a>
## Introduction

Kraken/Event component provides classes to implement event-based architecture.

<a name="introduction"></a>
## Features

Kraken/Event features:
<div class="dot-list" markdown="1">
- Both asynchronous and synchronous event emitters
- Event handlers
- Expanded interface for attaching listeners and managing events propagation
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

### Event-driven Architecture

Event-driven architecture is a software architecture pattern promoting applications and systems which transmit events among loosely coupled components and services. This kind of systems usually consists of event emitters and event listeners.

### Event Emitter

Event emitter is an object which purpose is to emit and transfer events.

### Event Listener

Event listener is an object which detects event presentation and reacts to it by calling corresponding event handler.

### Event

Event can be defined as a change in application state.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-base-emitters">
### Creating Base Emitters

    $emitter = new EventEmitter();

<a name="creating-asynchronous-emitters">
### Creating Asynchronous Emitters

    $emitter = new EventEmitter($loop); // $loop should be instance of \Kraken\Loop\Loop

<a name="emitting-events">
### Emitting Events

Emitting event notifies all listeners to call their corresponding callbacks. All callbacks are executed in the same order as they were attached. This method is blocking for base emitters and non-blocking for asynchronous ones.

    $emitter->emit('eventName', $eventParams = [ 'param1', 'param2' ]);

<a name="attaching-listeners">
### Attaching Listeners

Listeners attached via `on` method will be executed each time the event is presented.

    $emitter->on('eventName', function($param1, $param2) {
        // do something with event
    });

<a name="attaching-single-execution-listeners">
### Attaching Single Execution Listeners

Listeners attached via `once` method will be executed only once and after that the listener is unregistered automatically.

    $emitter->once('eventName', function() {
        // this handler will be executed only once
    });

<a name="attaching-multiple-execution-listeners">
### Attaching Multiple Execution Listeners

Listeners attached via `times` method will be executed as many times as second argument implies. After that, the listener is unregistered automatically.

    $emitter->times('eventName', 3, function() {
        // this handler will be executed exactly 3 times 
    });

<a name="delaying-listeners">
### Delaying Listeners

All of `on`, `once` and `times` events have a special `delay` prefixed alternatives that instead of being applied instantly, starts to be executed after specified amount of events have been emitted.

    $emitter->delayOnce('eventName', 2, function() {
        // this handler will be executed once when eventName is emitted 2nd time
    });
    
    $emitter->delayTimes('eventName', 2, 1, function() {
        // this works the same as previous example
    });

<a name="cancelling-listeners">
### Cancelling Listeners

All attached listeners can be cancelled, meaning they will be unregistered from event emitter.

It can be done via calling `cancel` method on handler you want to cancel.

    $handler = $emitter->on('eventName', $callback);
    $handler->cancel();

Cancelling can be also done indirectly using emitter.

    $emitter->on('eventName', $callback);
    $emitter->removeListener('eventName', $callback);

<a name="forwarding-and-discarding-events">
### Forwarding & Discarding Events

Events can be forwarded between two or more event emitters using `forwardEvents` method. Forwarding works in that way, when emitter A forwards its events to emitter B, then all listeners attached on B will receive event emitted on both A and B. It works only in one way, so event emitted on B won't be visible for A listeners.

    $emitterA = new EventEmitter();
    $emitterB = new EventEmitter();
    
    $emitterA->forwardEvents($emitterB);
    $emitterB->on('event', $callback);
    
    $emitterA->emit('event'); // this event will be forwarded (copied) to emitter B, and $callback will be executed.
    $emitterA->discardEvents($emitterB); // now longer any events will be forwarded

`forwardEvents` automatically passes all emitted events to its target. This might be not always desired behaviour. If you want to forward only specified set of events, then consider using `copyEvents` method.

    $emitterA = new EventEmitter();
    $emitterB = new EventEmitter();
    
    $emitterA->copyEvents($emitterB, [ 'event' ]); // only 'event' event will be forwarded.

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="async-event-emitter"></a>
### AsyncEventEmitter

    class AsyncEventEmitter implements EventEmitterInterface, LoopAwareInterface

AsyncEventEmitter is implementation of asynchronous event emitter.

<a name="base-event-emitter"></a>
### BaseEventEmitter

    class BaseEventEmitter implements EventEmitterInterface

BaseEventEmitter is implementation of synchronous event emitter.

<a name="event-emitter"></a>
### EventEmitter

    class EventEmitter implements EventEmitterInterface

EventEmitter is universal event emitter that by default behaves as synchronous emitter, but changes to asynchronous when provied with loop instance.

<a name="event-emitter-interface"></a>
### EventEmitterInterface

    interface EventEmitterInterface

<a name="event-handler"></a>
### EventHandler

    class EventHandler

EventHandler represents single event listening callback.
