# Loop Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Creating Loop](#creating-loop)
    - [Queuing](#queuing)
    - [Using Timers](#using-timers)
    - [Using Streams](#using-streams)
    - [Catching Throwables](#catching-throwables)
    - [Importing & Exporting](#importing-and-exporting)
    - [Using React Bridge](#using-react-bridge)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Loop](#loop)
    - [LoopInterface](#loop-interface)
    - [LoopExtendedInterface](#loop-extended-interface)
- [Important Directories](#important-directories)
    - [Bridge](#bridge)
    - [Model](#model)

<a name="introduction"></a>
## Introduction

Loop is a component that provides abstraction layer for writing asynchronous code in PHP on single thread or process with usage of single or multiple loop backgrounds.

<a name="introduction"></a>
## Features

Loop features:

<div class="dot-list" markdown="1">
- Interface for writing asynchronous code on single Thread or Process
- File descriptor polling
- One-time and periodic timers
- Deferred execution of callbacks
- Support for `stream_select` -based loops
- Support for `LibEvent` -based loops
- Support for `LibEv` -based loops
- Support for `ExtEvent` -based loops
- Support for using multiple loops with multiple execution flows
- Support for switching between loops and importing or exporting its unfinished queues
- ReactPHP compatibility
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Event-Loop

The event loop is a central part of [event-driven](/docs/{{version}}/events) application. It schedules listeners, runs timers, handles signals, and polls streams for pending reads and available writes. 

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-loop"></a>
### Creating Loop

To create loop you have to pass to its constructor one of models, that can be found in `Kraken\Loop\Model` namespace.

    $loop = new Loop(new SelectLoop);

<a name="queuing"></a>
### Queuing

To queue any callbacks use `onTick` method.

    $loop->onTick(function() {
        // this will be executed as soon as possible!
    });

For more control consider using `onBeforeTick` or `onAfterTick` methods.

<a name="using-timers"></a>
### Using Timers

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

To catch throwable throw by loop, use `try-catch` block. Kraken Framework uses this kind of structure for reacting to loop problems:

    while ($shouldProceed)
    {
        try
        {
            $loop->start();
        }
        catch (\Throwable $ex)
        {
            $shouldProceed = reactToProblem($ex);
        }
    }

<a name="importing-and-exporting"></a>
### Importing & Exporting

Importing and/or exporting the interal loop state might be done between two loop instances.

    $loop->export($loopBackup);

    $loop->import($loopBackup);

<a name="using-react-bridge"></a>
### Using React Bridge

To create ReactPHP loop using existing Kraken loop, use:

    $reactLoop = new \Kraken\Loop\Bridge\React\ReactLoop($krakenLoop);

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="loop"></a>
### Loop

    class Loop implements LoopExtendedInterface

Loop is an abstraction layer for any of models existing in `Kraken\Loop\Model` namespace.

<a name="loop-interface"></a>
### LoopInterface

    interface LoopInterface

<a name="loop-extended-interface"></a>
### LoopExtendedInterface

    interface LoopExtendedInterface extends LoopInterface

<a name="important-directories"></a>
## Important Directories

This section contains list of most important directories existing inside of component. It **does not include all** directories.

<a name="bridge"></a>
### Bridge

This folder contains all bridges allowing for creation of loops compatible with other loop-based systems or components.

<a name="model"></a>
### Model

This folder contains all models available to be injected to `Kraken\Loop\Loop` instance.
