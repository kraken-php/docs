# Throwable Component

- [Introduction](#introduction)
- [Features](#features)
- [Concepts](#concepts)
- [Examples](#examples)
    - [Throwable Hierarchy](#throwable-hierarchy)
    - [Reading Throwable Traces](#reading-throwable-traces)
- [Important Classes & Interfaces](#important-classes-and-interfaces)
    - [Error](#error)
    - [Exception](#exception)
    - [Throwable](#throwable)
    - [ThrowableProxy](#throwable-proxy)

<a name="introduction"></a>
## Introduction

Throwable is a component that provides throwable hierarchy used in Kraken Framework and additional helpers.

<a name="introduction"></a>
## Features

Throwable features:

<div class="dot-list" markdown="1">
- Custom hierarchy of Throwables created in mind of unifying error and exception handling in PHP5 and PHP7
- Custom format for stack trace
- Support for chaining Throwables
- Built-in ErrorHandler
- Built-in ExceptionHandler
- Implementation of throwable objects proxy
- Kraken Framework compatibility
</div>

<a name="concepts"></a>
## Concepts

This section contains terminology, useful concepts and definitions, that might be helpful to learn to fully understand this component purpose.

### Throwable

Throwable is the base interface for any object that can be thrown via a throw statement in PHP 7.0, including Error and Exception. 

### Error

Error is the base class for all internal PHP errors. 

### Exception

Exception is the base class for all Exceptions in PHP 5, and the base class for all user exceptions in PHP 7.

<a name="examples"></a>
## Examples

This section contains examples and patterns that can be used with described component.

<a name="throwable-hierarchy"></a>
### Throwable Hierarchy

Kraken uses its own custom made Throwable hierarchy.

    Throwable
        Error
            FatalError
            WarningError
            NoticeError
        Exception
            Logic
                IllegalCallException
                IllegalFieldException
                InstantiationException
                InvalidArgumentException
                InvalidFormatException
                ResourceException
                ResourceOccupiedException
                ResourceUndefinedException
            Runtime
                CancellationException
                ExecutionException
                OutOfBoundsException
                OverflowException
                ReadException
                RejectionException
                TimeoutException
                UnderflowException
                WriteException
            System
                ChildUnresponsiveException
                ParentUnresponsiveException
                TaskIncompleteException
            LogicException
            RuntimeException
            SystemException

<a name="reading-throwable-trace"></a>
### Reading Throwable Traces

To read the full trace of any Throwable (it does not need to be part of Kraken's throwable hierarchy) use:

    // for exceptions
    $trace = \Kraken\Throwable\Exception::toTrace($exception);
    
    // for errors
    $trace = \Kraken\Throwable\Error::toTrace($error);

<a name="important-classes-and-interfaces"></a>
## Important Classes & Interfaces

This section contains list of most important classes and interfaces shipped with this component. It **does not include all** classes and interface.

<a name="error"></a>
### Error

    class Error extends \Error

Error is Kraken implementation of PHP7.0 `\Error` class.

<a name="exception"></a>
### Exception

    class Exception extends \Exception

Exception is Kraken implementation of PHP `\Exception` class.

<a name="throwable"></a>
### Throwable

    abstract class Throwable

Throwable is composition of static methods to operate on `\Throwable`.

<a name="throwable-proxy"></a>
### ThrowableProxy

    class ThrowableProxy

ThrowableProxy is special class allows creation of proxies for `\Throwable` objects. This class should be used in design patterns which logic represents failures as throwables, and does not necessarily need stack information.

> {notice} Throwables were designed to handle exceptional states that can occur in application during execution. They were not meant to be used in business logic, however in practice, many design patterns emerged that used Throwables as a type representation of failure, for which application needed to react. In other programming languages Throwables are populated with data not on creation but while throwing. In PHP it works differently, and Throwables are populated on creation. This can lead to major problems with memory and performance with usage of previously mentioned design patterns. In those cases the valuable pieces of information are Throwable class, message, code and previous element, but not stack trace that requires most of memory allocation. For this exclusive need ThrowableProxy has been created. Its main purpose is to create a placeholders for Throwable most needed data discarding all not needed traces.
