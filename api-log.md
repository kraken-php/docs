# Log Component

- [Introduction](#introduction)
- [Features](#features)
- [Examples](#examples)
    - [Creating Logger](#creating-logger)
    - [Logging](#logging)
    - [Creating Monolog Handlers](#creating-monolog-handlers)
    - [Creating Monolog Processors](#creating-monolog-processors)
    - [Creating Monolog Formatters](#creating-monolog-formatters)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Logger](#logger)
    - [LoggerInterface](#logger-interface)
    - [LoggerFactory](#logger-factory)

<a name="introduction"></a>
## Introduction

Log is a component that allows writing application logs using files, sockets, inboxes, databases and various web services.

<a name="introduction"></a>
## Features

Log features:

<div class="dot-list" markdown="1">
- Support for storing logs in files, databases or cloud storages
- Decorators for monolog formatters
- Decorators for monolog handlers
- Decorators for monolog processors
- PSR-3 compatibility
- Kraken Framework compatibility
</div>

## Examples

This section contains examples and patterns that can be used with described component.

<a name="creating-logger"></a>
### Creating Logger

To create logger use this syntax:

    $logger = new Logger($loggerName, $loggers, $processors);

<a name="logging"></a>
### Logging

Logging uses the same interface as PSR-3 or Monolog, so you can use methods such as `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, `debug` and others.

The `log` method is also supported:

    $logger->log($logger::WARNING, 'Something went wrong!', $context);

<a name="creating-monolog-handlers"></a>
### Creating Monolog Handlers

Creating monolog handlers wrapped in Kraken-compatible interface can be done using:

    $handler = $loggerFactory->createHandler($handlerNameOrClass, $handlerArgs);

<a name="creating-monolog-processors"></a>
### Creating Monolog Processors

Creating monolog processors wrapped in Kraken-compatible interface can be done using:

    $processor = $loggerFactory->createProcessor($processorNameOrClass, $processorArgs);

<a name="creating-monolog-formatters"></a>
### Creating Monolog Formatters

Creating monolog formatters wrapped in Kraken-compatible interface can be done using:

    $formatter = $loggerFactory->createFormatter($formatterNameOrClass, $formatterArgs);

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="logger"></a>
### Logger

    class Logger implements LoggerInterface

It is an implementation of logger using monolog logger as its model.

<a name="logger-interface"></a>
### LoggerInterface

    interface LoggerInterface extends EnumInterface, PsrLoggerInterface
 
PSR-3 compatible interface for logging.

<a name="logger-factory"></a>
### LoggerFactory

    class LoggerFactory
 
Factory for creating Kraken-compatible instances of Monolog handlers, processors and formatters.
