# Asynchronity

- [Introduction](#introduction)
- [Definitions](#definitions)
    - [Synchronous Programming](#synchronous-programming)
    - [Asynchronous Programming](#asynchronous-programming)
- [Implementation](#implementation)
    - [Events](#events)
    - [Messages](#messages)
    - [Promises](#promises)
    - [Event-Loop](#event-loop)

<a name="introduction"></a>
## Introduction

One of the ideas behind Kraken Framework is to provide mechanisms for asynchronous programming in PHP. Code asynchronity and asynchronous programming might be a difficult concept to understand and to apply to real code. This article shows important concepts about this kind of programming and how they have been implemented in Kraken Framework.

<a name="definitions"></a>
## Definitions

This section presents definitions for synchronous and asynchronous programming and shows differences between them.

<a name="synchronous-programming"></a>
### Synchronous Programming

A method call is considered synchronous or sequential if the caller cannot make progress until the method returns a value or throws an exception. This simply means, that no matter what kinds of operations you are doing in your program, each operation is executed one after another, but only when previous operation finishes.

This can become problematic quite fast as your application will start to involve using blocking operations such as sending or receiving data from external sources outside of your system. A common example for this kind of behaviour is transmitting data through sockets. Data being read from a socket is not immediately available and it may take some time for the data to be delivered. In synchronous programming this intervals of time makes your application to freeze and wait for external program response doing nothing and killing its overall performance.
 
Consider this simple example of reading data using socket:

    $socket = stream_socket_client(
        $endpoint, $errno, $errstr, $timeout, STREAM_CLIENT_CONNECT, $context
    );
    
    $data = fread($socket);
    doSmth($data);
    fclose($socket);
    
    // do other things

Both `stream_socket_client` and `fread` operations used in this example are synchronous and blocking. It means the `do other things` block won't be executed before these functions return. Think what would happen if for some reason your application had to wait one hour for this socket to receive data? Do you see the problem? Yes, your application would be frozen, for whole hour, doing nothing.

This kind of problems might be omitted using asynchronous programming.

<a name="asynchronous-programming"></a>
### Asynchronous Programming

In contrast to synchronous programming, an asynchronous call allows the caller to only initiate an operation which completion will be signalled via some additional mechanism. This means, that using previously shown example, your application could initiate the socket listener, and then do other things, while data is being delivered.

Let's improve previous example using Kraken's `Kraken\Loop\Loop`:

    $socket = stream_socket_client(
        $endpoint, $errno, $errstr, $timeout, STREAM_CLIENT_CONNECT, $context
    );
    
    $loop->addReadStream($socket, function($socket) {
        $data = fread($socket);
        doSmth($data);
        fclose($socket);
    });
    
    // do other things

Or, even better, using Kraken's asynchronous `Kraken\Ipc\Socket\Socket`:

    $socket = new Socket($endpoint, $loop);
    
    $socket->on('data', function($socket, $data) {
        doSmth($data);
        $socket->close();
    });
    
    // do other things

As you can see, after these additions, your code doesn't try to read data automatically, but only registers data listeners allowing `do other things` to be processed immediately. Kraken's internal mechanisms will ensure that your registered listener will be executed as soon as data is received. Now, your application can freely do other things, not wasting any precious server resources. This kind of an approach in creating applications is referenced to, as [event-driven programming](/docs/{{version}}/events).

<a name="implementation"></a>
## Implementation

Kraken Framework allows developer to write asynchronous code using [events](#events), [messages](#messages), [promises](#promises) and [event-loop](#event-loop).

<a name="events"></a>
### Events

Events allow you to write loose-coupled services that communicates between themselves using transfer of special signals called events. Kraken implements event-driven architecture using [Kraken/Event](/docs/{{version}}/api-event) component. You can read more about how events can be used in [events section](/docs/{{version}}/events#emitting-and-listening).

<a name="messages"></a>
### Messages

Messages can be defined as special case of events, that instead of local use, are passed between isolated processes or threads. Kraken implements message-driven architecture using [Kraken/Channel](/docs/{{version}}/api-channel) component. You can read more about how messages can be used in [messages section](/docs/{{version}}/messages).

<a name="promises"></a>
### Promises

Promises represent the eventual results of an asynchronous operations. They allow writing asynchronous code in synchronous-like manner. Kraken implements promises using [Kraken/Promise](/docs/{{version}}/api-promise) component. You can read more how promises can be used in [promises section](/docs/{{version}}/promises).

<a name="event-loop"></a>
### Event-Loop

Event loop is central part of any Kraken-based application. It is an infinite loop that queues and resolves callbacks allowing for asynchronous processing on single thread. Kraken implements loop using [Kraken/Loop](/docs/{{version}}/api-loop) component. You can read more about how loop can be used in [loop section](/docs/{{version}}/events#event-loop).
